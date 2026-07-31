# Report 004 — PlatformAppV0 API admin exchange 401 investigation

## 1. Verdict

`BLOCKED_PLATFORMAPPV0_API_ADMIN_EXCHANGE_401`

Source correction was implemented to expose the true failure and to stop returning a generic `PLATFORM_ADMIN_NOT_AUTHENTICATED`, but physical admin authorization cannot pass yet because the running ApiServer PlatformAppV0 Phase1 profile still reports:

```text
HTTP 401
resultCode = GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

The next operator action is configuration, not another UI hydration fix.

## 2. Reproduction exact URL/result code

User-observed URL before prompt004:

```text
https://localhost:7012/?admin_authorize_result=HTTP_401_PLATFORM_ADMIN_NOT_AUTHENTICATED
```

Prompt004 synthetic runtime proof after correction:

```text
POST http://127.0.0.1:7161/api/platform-v0/admin/google/exchange
Body field present: idToken
Body token: dummy non-secret value
HTTP 401
resultCode = GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

This proves the previous generic 401 has been split and the current physical blocker is missing ApiServer Google ClientId configuration for token audience validation.

## 3. Exact request/response timeline

```text
PlatformAppV0 /platform-admin/authorize
-> reads saved OIDC id_token from encrypted app cookie
-> PlatformAppV0ApiClient POSTs JSON body to:
   http://127.0.0.1:7161/api/platform-v0/admin/google/exchange
-> request DTO:
   { "idToken": "<Google ID token>" }
-> ApiServer controller is [AllowAnonymous]
-> controller checks token presence
-> controller checks PlatformAppV0Options.GoogleClientId
-> current runtime: GoogleClientId is absent
-> controller returns HTTP 401 GOOGLE_CLIENT_ID_NOT_CONFIGURED
-> PlatformAppV0 preserves resultCode in redirect
```

## 4. Exact failure location

```text
Failure file: E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs
Failure type/method: PlatformAppV0Phase1Controller.ExchangeGoogle
Failure condition: string.IsNullOrWhiteSpace(options.Value.GoogleClientId)
Returned HTTP status: 401
Returned resultCode before prompt004: PLATFORM_ADMIN_NOT_AUTHENTICATED
Returned resultCode after prompt004: GOOGLE_CLIENT_ID_NOT_CONFIGURED
Why condition was true: runtime synthetic proof still returns GOOGLE_CLIENT_ID_NOT_CONFIGURED after restart, so the ApiServer Phase1 process has no effective Google client id loaded.
```

## 5. Root cause

The admin exchange endpoint validates the Google ID token audience server-side. That is correct: the Platform API, not the UI, must approve administrator identity.

The ApiServer Phase1 runtime did not have an effective Google ClientId in `PlatformAppV0Options.GoogleClientId`. Before prompt004, this branch returned the same generic result as a missing/invalid token:

```text
PLATFORM_ADMIN_NOT_AUTHENTICATED
```

That hid the true cause. Prompt004 changed the code so the same runtime failure is now:

```text
GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

Secondary cause found: the phase1-only branch in `ApiServer01\Program.cs` returns before the later `AddUserSecrets(...)` branch, so PlatformAppV0 phase1 configuration could miss user secrets unless the module loads or receives them itself.

## 6. Why prompt003 did not solve this

Prompt003 fixed the PlatformAppV0 browser/cookie/UI hydration path. The user result after prompt003 included `HTTP_401_...`, proving `/platform-admin/authorize` was called and the API exchange was reached. Therefore the remaining failure moved from UI/cookie hydration to API exchange configuration and validation.

## 7. Exact endpoint/method/request contract

```text
Method: POST
Path: /api/platform-v0/admin/google/exchange
Request DTO: GoogleExchangeRequest
Field: IdToken, serialized as idToken
Response DTO: GoogleExchangeResponse on success
Problem DTO: PlatformAppV0ProblemResponse on failure
Authentication: [AllowAnonymous] at ASP.NET middleware layer
Credential: Google ID token in JSON body
```

The exchange endpoint is intentionally anonymous because the Google ID token is the credential being validated. It does not grant Platform admin access until Google validation and allowlist matching pass.

## 8. Authentication/authorization scheme analysis

- `POST /api/platform-v0/admin/google/exchange` is `[AllowAnonymous]`.
- It is not protected by `PlatformAppV0Admin` bearer policy.
- `PlatformAppV0Admin` bearer policy protects later admin actions such as Tenant/POS creation.
- Successful Google exchange returns a separate Platform administrator JWT.
- PlatformAppV0 persists approved admin proof only after HTTP 200 and `PLATFORM_ADMIN_AUTHENTICATED`.

## 9. Google token validation analysis

Safe validation requirements:

- token must be present;
- ApiServer must have expected Google ClientId configured;
- Google validator must validate issuer, audience, expiration, signature and token structure;
- email must be verified;
- subject or normalized email must match approved admin bootstrap configuration.

Prompt004 implemented result-code separation for:

```text
GOOGLE_ID_TOKEN_NOT_SENT
GOOGLE_CLIENT_ID_NOT_CONFIGURED
GOOGLE_ID_TOKEN_INVALID_ISSUER
GOOGLE_ID_TOKEN_INVALID_AUDIENCE
GOOGLE_ID_TOKEN_EXPIRED
GOOGLE_ID_TOKEN_VALIDATION_FAILED
GOOGLE_EMAIL_NOT_VERIFIED
PLATFORM_ADMIN_BOOTSTRAP_IDENTITY_NOT_FOUND
PLATFORM_ADMIN_IDENTITY_MISMATCH
```

No raw Google token or claim payload was printed.

## 10. Platform admin bootstrap/allowlist analysis

The API permits administrator approval only when:

```text
ApprovedAdminEmail matches normalized Google email
OR
ApprovedGoogleSubject matches Google subject
```

If neither approved identity config exists:

```text
HTTP 403
PLATFORM_ADMIN_BOOTSTRAP_IDENTITY_NOT_FOUND
```

If configured but not matching:

```text
HTTP 403
PLATFORM_ADMIN_IDENTITY_MISMATCH
```

## 11. Relevant C# evidence

### 11.1 PlatformAppV0 authorize endpoint

Full local path:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Program.cs
```

Project-relative path:

```text
PlatformAppV0/src/OBM.PlatformAppV0/Program.cs
```

Class/type name: top-level `Program`  
Method/endpoint: `MapGet("/platform-admin/authorize", ...)`  
Approximate line range: 75-146  
Role: reads OIDC id_token, calls API exchange, writes approved admin claims after success.

```csharp
app.MapGet("/platform-admin/authorize", async (
    HttpContext context,
    PlatformAppV0ApiClient api,
    CancellationToken cancellationToken) =>
{
    var auth = await context.AuthenticateAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    var principal = auth.Principal ?? context.User;
    if (auth.Succeeded != true || principal.Identity?.IsAuthenticated != true)
    {
        context.Response.Redirect("/?admin_authorize_result=PLATFORM_ADMIN_NOT_AUTHENTICATED");
        return;
    }

    var idToken = auth.Properties?.GetTokenValue(OpenIdConnectParameterNames.IdToken) ??
        await context.GetTokenAsync(CookieAuthenticationDefaults.AuthenticationScheme, OpenIdConnectParameterNames.IdToken);
    if (string.IsNullOrWhiteSpace(idToken))
    {
        context.Response.Redirect($"/?admin_authorize_result={PlatformAppV0Contract.GoogleIdTokenUnavailable}");
        return;
    }

    var exchange = await api.ExchangeGoogleWithStatusAsync(idToken, cancellationToken);
    if (exchange.HttpStatusCode < StatusCodes.Status200OK || exchange.HttpStatusCode >= StatusCodes.Status300MultipleChoices)
    {
        var resultCode = exchange.ResultCode ?? PlatformAppV0Contract.AdminExchangeHttpFailed;
        context.Response.Redirect($"/?admin_authorize_result=HTTP_{exchange.HttpStatusCode}_{Uri.EscapeDataString(resultCode)}");
        return;
    }

    if (exchange.Body?.Success != true ||
        exchange.Body.ResultCode != PlatformAppV0Contract.AdminAuthenticated)
    {
        var resultCode = exchange.ResultCode ?? PlatformAppV0Contract.AdminExchangeContractFailed;
        context.Response.Redirect($"/?admin_authorize_result={Uri.EscapeDataString(resultCode)}");
        return;
    }

    if (string.IsNullOrWhiteSpace(exchange.Body.AdministratorToken) ||
        exchange.Body.Administrator is null ||
        string.IsNullOrWhiteSpace(exchange.Body.Administrator.NormalizedEmail))
    {
        context.Response.Redirect($"/?admin_authorize_result={PlatformAppV0Contract.ApprovedIdentityMissing}");
        return;
    }

    var claims = principal.Claims
        .Where(claim => !claim.Type.StartsWith("obm_platform_admin_", StringComparison.Ordinal))
        .ToList();
    claims.Add(new Claim(PlatformAppV0Contract.AdminTokenClaim, exchange.Body.AdministratorToken));
    claims.Add(new Claim(PlatformAppV0Contract.AdminUserGuidClaim, exchange.Body.Administrator.PlatformAdminUserGuid.ToString()));
    claims.Add(new Claim(PlatformAppV0Contract.AdminEmailClaim, exchange.Body.Administrator.NormalizedEmail));
    claims.Add(new Claim(PlatformAppV0Contract.AdminDisplayNameClaim, exchange.Body.Administrator.DisplayName));

    var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme, ClaimTypes.Name, ClaimTypes.Role);
    await context.SignInAsync(
        CookieAuthenticationDefaults.AuthenticationScheme,
        new ClaimsPrincipal(identity),
        auth.Properties);
    context.Response.Redirect("/");
});
```

### 11.2 PlatformAppV0 API client

Full local path:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Application\PlatformAppV0ApiClient.cs
```

Project-relative path:

```text
PlatformAppV0/src/OBM.PlatformAppV0/Application/PlatformAppV0ApiClient.cs
```

Class/type name: `PlatformAppV0ApiClient`  
Method: `ExchangeGoogleWithStatusAsync`  
Approximate line range: 17-31  
Role: sends exact request DTO and preserves API status/resultCode.

```csharp
public async Task<PlatformAppV0ApiCall<GoogleExchangeResponse>> ExchangeGoogleWithStatusAsync(string idToken, CancellationToken cancellationToken = default)
{
    var response = await Client.PostAsJsonAsync($"{_baseUrl}/api/platform-v0/admin/google/exchange", new GoogleExchangeRequest(idToken), cancellationToken);
    GoogleExchangeResponse? body = null;
    try
    {
        body = await response.Content.ReadFromJsonAsync<GoogleExchangeResponse>(cancellationToken);
    }
    catch (Exception)
    {
        return new PlatformAppV0ApiCall<GoogleExchangeResponse>((int)response.StatusCode, null, "RESPONSE_DESERIALIZATION_FAILED");
    }

    return new PlatformAppV0ApiCall<GoogleExchangeResponse>((int)response.StatusCode, body, body?.ResultCode);
}

public sealed record PlatformAppV0ApiCall<T>(int HttpStatusCode, T? Body, string? ResultCode);
```

### 11.3 Exchange DTOs and result codes

Full local path:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts\PlatformAppV0Contracts.cs
```

Project-relative path:

```text
PlatformAppV0/src/OBM.PlatformAppV0.Contracts/PlatformAppV0Contracts.cs
```

Type name: `PlatformAppV0Contract` and records  
Approximate line range: 3-66  
Role: shared contract/result codes.

```csharp
public static class PlatformAppV0Contract
{
    public const string Version = "platform-app-v0.phase1";
    public const string AdminAuthenticated = "PLATFORM_ADMIN_AUTHENTICATED";
    public const string PlatformAdminNotAuthenticated = "PLATFORM_ADMIN_NOT_AUTHENTICATED";
    public const string PlatformAdminNotAuthorized = "PLATFORM_ADMIN_NOT_AUTHORIZED";
    public const string GoogleIdTokenNotSent = "GOOGLE_ID_TOKEN_NOT_SENT";
    public const string GoogleClientIdNotConfigured = "GOOGLE_CLIENT_ID_NOT_CONFIGURED";
    public const string GoogleIdTokenInvalidIssuer = "GOOGLE_ID_TOKEN_INVALID_ISSUER";
    public const string GoogleIdTokenInvalidAudience = "GOOGLE_ID_TOKEN_INVALID_AUDIENCE";
    public const string GoogleIdTokenExpired = "GOOGLE_ID_TOKEN_EXPIRED";
    public const string GoogleIdTokenValidationFailed = "GOOGLE_ID_TOKEN_VALIDATION_FAILED";
    public const string GoogleEmailNotVerified = "GOOGLE_EMAIL_NOT_VERIFIED";
    public const string PlatformAdminBootstrapIdentityNotFound = "PLATFORM_ADMIN_BOOTSTRAP_IDENTITY_NOT_FOUND";
    public const string PlatformAdminIdentityMismatch = "PLATFORM_ADMIN_IDENTITY_MISMATCH";
    public const string GoogleIdTokenUnavailable = "GOOGLE_ID_TOKEN_UNAVAILABLE";
    public const string AdminExchangeHttpFailed = "ADMIN_EXCHANGE_HTTP_FAILED";
    public const string AdminExchangeContractFailed = "ADMIN_EXCHANGE_CONTRACT_FAILED";
    public const string ApprovedIdentityMissing = "APPROVED_IDENTITY_MISSING";
    public const string AdminScheme = "PlatformAppV0Admin";
    public const string AdminPolicy = "PlatformAppV0Admin";
    public const string AdminTokenClaim = "obm_platform_admin_token";
    public const string AdminUserGuidClaim = "obm_platform_admin_user_guid";
    public const string AdminEmailClaim = "obm_platform_admin_email";
    public const string AdminDisplayNameClaim = "obm_platform_admin_display_name";
}

public sealed record PlatformAppV0ProblemResponse(
    bool Success,
    string ResultCode,
    string ContractVersion,
    string CorrelationId);

public sealed record GoogleExchangeRequest(string IdToken);

public sealed record GoogleExchangeResponse(
    bool Success,
    string ResultCode,
    string ContractVersion,
    string CorrelationId,
    string? AdministratorToken,
    PlatformAdminIdentityResponse? Administrator);
```

### 11.4 API controller exchange endpoint

Full local path:

```text
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs
```

Project-relative path:

```text
1ApiServer/ApiServer01/PlatformAppV0/Controllers/PlatformAppV0Phase1Controller.cs
```

Class/type name: `PlatformAppV0Phase1Controller`  
Method: `ExchangeGoogle`  
Approximate line range: 13-85  
Role: validates Google token and admin allowlist, issues Platform admin token.

```csharp
[ApiController]
[Route("api/platform-v0")]
public sealed class PlatformAppV0Phase1Controller(
    IPlatformAppV0Store store,
    IPairingCodeProtector pairingCodes,
    IPlatformAppV0TokenService tokens,
    IGoogleAdminIdentityValidator googleIdentity,
    IOptions<PlatformAppV0Options> options) : ControllerBase
{
    [HttpPost("admin/google/exchange")]
    [AllowAnonymous]
    public async Task<IActionResult> ExchangeGoogle([FromBody] GoogleExchangeRequest request, CancellationToken cancellationToken)
    {
        var correlationId = CorrelationId();
        if (string.IsNullOrWhiteSpace(request.IdToken))
        {
            return StatusCode(StatusCodes.Status401Unauthorized, Problem(PlatformAppV0Contract.GoogleIdTokenNotSent, correlationId));
        }

        if (string.IsNullOrWhiteSpace(options.Value.GoogleClientId))
        {
            return StatusCode(StatusCodes.Status401Unauthorized, Problem(PlatformAppV0Contract.GoogleClientIdNotConfigured, correlationId));
        }

        var google = await googleIdentity.ValidateAsync(request.IdToken, options.Value.GoogleClientId, cancellationToken);
        if (!google.Success)
        {
            return StatusCode(StatusCodes.Status401Unauthorized, Problem(google.ResultCode, correlationId));
        }

        if (!google.EmailVerified)
        {
            return StatusCode(StatusCodes.Status403Forbidden, Problem(PlatformAppV0Contract.GoogleEmailNotVerified, correlationId));
        }

        var email = NormalizeEmail(google.Email);
        var approvedEmail = NormalizeEmail(options.Value.ApprovedAdminEmail);
        var approvedSubject = options.Value.ApprovedGoogleSubject?.Trim();
        var emailMatches = !string.IsNullOrWhiteSpace(approvedEmail) && email == approvedEmail;
        var subjectMatches = !string.IsNullOrWhiteSpace(approvedSubject) && google.Subject == approvedSubject;
        if (string.IsNullOrWhiteSpace(approvedEmail) && string.IsNullOrWhiteSpace(approvedSubject))
        {
            return StatusCode(StatusCodes.Status403Forbidden, Problem(PlatformAppV0Contract.PlatformAdminBootstrapIdentityNotFound, correlationId));
        }

        if (!emailMatches && !subjectMatches)
        {
            return StatusCode(StatusCodes.Status403Forbidden, Problem(PlatformAppV0Contract.PlatformAdminIdentityMismatch, correlationId));
        }

        var now = DateTimeOffset.UtcNow;
        var admin = await store.UpdateAsync(state =>
        {
            var existing = state.AdminUsers.FirstOrDefault(x =>
                string.Equals(x.NormalizedEmail, email, StringComparison.OrdinalIgnoreCase) ||
                (!string.IsNullOrWhiteSpace(google.Subject) && x.GoogleSubject == google.Subject));
            if (existing is not null)
            {
                var updated = existing with { GoogleSubject = google.Subject, DisplayName = google.DisplayName ?? existing.DisplayName, IsActive = true, UpdatedAtUtc = now };
                state.AdminUsers[state.AdminUsers.IndexOf(existing)] = updated;
                return updated;
            }

            var created = new PlatformAppV0AdminUser(Guid.NewGuid(), google.Subject, email, google.DisplayName ?? email, true, now, now);
            state.AdminUsers.Add(created);
            return created;
        }, cancellationToken);

        var identity = new PlatformAdminIdentityResponse(true, PlatformAppV0Contract.AdminAuthenticated, PlatformAppV0Contract.Version, correlationId, admin.PlatformAdminUserGuid, admin.NormalizedEmail, admin.DisplayName, admin.IsActive);
        return Ok(new GoogleExchangeResponse(true, PlatformAppV0Contract.AdminAuthenticated, PlatformAppV0Contract.Version, correlationId, tokens.CreateAdminToken(admin), identity));
    }

    private static PlatformAppV0ProblemResponse Problem(string resultCode, string correlationId) =>
        new(false, resultCode, PlatformAppV0Contract.Version, correlationId);
}
```

### 11.5 API module auth/options registration

Full local path:

```text
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\PlatformAppV0Module.cs
```

Project-relative path:

```text
1ApiServer/ApiServer01/PlatformAppV0/PlatformAppV0Module.cs
```

Class/type name: `PlatformAppV0Module`  
Method: `AddPlatformAppV0Phase1`  
Approximate line range: 11-65  
Role: options binding, user-secret fallback, DI, JWT schemes.

```csharp
public static IServiceCollection AddPlatformAppV0Phase1(this IServiceCollection services, IConfiguration configuration)
{
    services.AddOptions<PlatformAppV0Options>()
        .Bind(configuration.GetSection(PlatformAppV0Options.SectionName))
        .PostConfigure(options =>
        {
            options.GoogleClientId = FirstConfigured(options.GoogleClientId, configuration["Authentication:Google:ClientId"], UserSecret("Authentication:Google:ClientId"));
            options.ApprovedAdminEmail = FirstConfigured(options.ApprovedAdminEmail, configuration["Authentication:Google:ApprovedAdminEmail"], UserSecret("Authentication:Google:ApprovedAdminEmail"));
            options.ApprovedGoogleSubject = FirstConfigured(options.ApprovedGoogleSubject, configuration["Authentication:Google:ApprovedGoogleSubject"], UserSecret("Authentication:Google:ApprovedGoogleSubject"));
        });
    services.AddSingleton<IPlatformAppV0Store, JsonPlatformAppV0Store>();
    services.AddSingleton<IPairingCodeProtector, PairingCodeProtector>();
    services.AddSingleton<IPlatformAppV0TokenService, PlatformAppV0TokenService>();
    services.AddSingleton<IGoogleAdminIdentityValidator, GoogleAdminIdentityValidator>();

    var options = configuration.GetSection(PlatformAppV0Options.SectionName).Get<PlatformAppV0Options>() ?? new PlatformAppV0Options();
    options.GoogleClientId = FirstConfigured(options.GoogleClientId, configuration["Authentication:Google:ClientId"], UserSecret("Authentication:Google:ClientId"));
    options.ApprovedAdminEmail = FirstConfigured(options.ApprovedAdminEmail, configuration["Authentication:Google:ApprovedAdminEmail"], UserSecret("Authentication:Google:ApprovedAdminEmail"));
    options.ApprovedGoogleSubject = FirstConfigured(options.ApprovedGoogleSubject, configuration["Authentication:Google:ApprovedGoogleSubject"], UserSecret("Authentication:Google:ApprovedGoogleSubject"));
    services.AddAuthentication()
        .AddJwtBearer(PlatformAppV0Contract.AdminScheme, jwt =>
        {
            jwt.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidIssuer = options.AdminIssuer,
                ValidateAudience = true,
                ValidAudience = options.AdminAudience,
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(PlatformAppV0TokenService.ResolveKey(options.AdminJwtSigningKey, "admin")),
                ValidateLifetime = true,
                ClockSkew = TimeSpan.FromMinutes(1)
            };
        })
        .AddJwtBearer(PlatformAppV0Contract.WpfJwtScheme, jwt => { /* WPF JWT validation omitted from admin exchange report */ });

    return services;
}

private static string? FirstConfigured(params string?[] values) =>
    values.FirstOrDefault(value => !string.IsNullOrWhiteSpace(value));

private static string? UserSecret(string key) =>
    new ConfigurationBuilder()
        .AddUserSecrets(Assembly.GetExecutingAssembly(), optional: true)
        .Build()[key];
```

### 11.6 Google ID-token validator

Full local path:

```text
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Security\GoogleAdminIdentityValidator.cs
```

Project-relative path:

```text
1ApiServer/ApiServer01/PlatformAppV0/Security/GoogleAdminIdentityValidator.cs
```

Type name: `GoogleAdminIdentityValidator`  
Method: `ValidateAsync`  
Approximate line range: 1-59  
Role: validates Google ID token and maps failures to safe result codes.

```csharp
public interface IGoogleAdminIdentityValidator
{
    Task<GoogleAdminIdentityValidationResult> ValidateAsync(string idToken, string expectedAudience, CancellationToken cancellationToken = default);
}

public sealed class GoogleAdminIdentityValidator : IGoogleAdminIdentityValidator
{
    public async Task<GoogleAdminIdentityValidationResult> ValidateAsync(string idToken, string expectedAudience, CancellationToken cancellationToken = default)
    {
        try
        {
            var payload = await GoogleJsonWebSignature.ValidateAsync(idToken, new GoogleJsonWebSignature.ValidationSettings
            {
                Audience = [expectedAudience]
            });

            return GoogleAdminIdentityValidationResult.Valid(
                payload.Subject,
                payload.Email,
                payload.EmailVerified == true,
                payload.Name);
        }
        catch (Exception ex)
        {
            return GoogleAdminIdentityValidationResult.Invalid(Classify(ex));
        }
    }

    private static string Classify(Exception ex)
    {
        var message = ex.Message;
        if (message.Contains("issuer", StringComparison.OrdinalIgnoreCase))
        {
            return PlatformAppV0Contract.GoogleIdTokenInvalidIssuer;
        }

        if (message.Contains("audience", StringComparison.OrdinalIgnoreCase) ||
            message.Contains("Invalid JWT: Wrong recipient", StringComparison.OrdinalIgnoreCase))
        {
            return PlatformAppV0Contract.GoogleIdTokenInvalidAudience;
        }

        if (message.Contains("expired", StringComparison.OrdinalIgnoreCase) ||
            message.Contains("Expiration", StringComparison.OrdinalIgnoreCase))
        {
            return PlatformAppV0Contract.GoogleIdTokenExpired;
        }

        return PlatformAppV0Contract.GoogleIdTokenValidationFailed;
    }
}
```

### 11.7 Focused tests

Full local path:

```text
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0\PlatformAppV0Phase1Tests.cs
```

Project-relative path:

```text
1ApiServer/ApiServer01.Tests/PlatformAppV0/PlatformAppV0Phase1Tests.cs
```

Type name: `PlatformAppV0Phase1Tests`  
Approximate role: proves anonymous exchange endpoint, options fallback, missing token, invalid issuer/audience/expired, email verification, missing/mismatched approved identity, and successful approved exchange.

```csharp
[Fact]
public void AdminExchange_IsAnonymousAndNotBlockedByAdminPolicy()
{
    var method = typeof(PlatformAppV0Phase1Controller).GetMethod(nameof(PlatformAppV0Phase1Controller.ExchangeGoogle));

    Assert.NotNull(method);
    Assert.NotNull(method!.GetCustomAttributes(typeof(AllowAnonymousAttribute), inherit: false).SingleOrDefault());
    Assert.Empty(method.GetCustomAttributes(typeof(AuthorizeAttribute), inherit: false));
}

[Fact]
public void Options_FallbackToCanonicalGoogleAuthenticationKeys()
{
    var configuration = new ConfigurationBuilder()
        .AddInMemoryCollection(new Dictionary<string, string?>
        {
            ["Authentication:Google:ClientId"] = "google-client-id",
            ["Authentication:Google:ApprovedAdminEmail"] = "admin@example.test",
            ["Authentication:Google:ApprovedGoogleSubject"] = "google-subject"
        })
        .Build();
    var services = new ServiceCollection();

    services.AddPlatformAppV0Phase1(configuration);
    using var provider = services.BuildServiceProvider();
    var options = provider.GetRequiredService<IOptions<PlatformAppV0Options>>().Value;

    Assert.Equal("google-client-id", options.GoogleClientId);
    Assert.Equal("admin@example.test", options.ApprovedAdminEmail);
    Assert.Equal("google-subject", options.ApprovedGoogleSubject);
}

[Theory]
[InlineData(PlatformAppV0Contract.GoogleIdTokenInvalidIssuer)]
[InlineData(PlatformAppV0Contract.GoogleIdTokenInvalidAudience)]
[InlineData(PlatformAppV0Contract.GoogleIdTokenExpired)]
public async Task AdminExchange_InvalidTokenFailsExplicit(string resultCode)
{
    var result = await Controller(validator: new FakeGoogleValidator(GoogleAdminIdentityValidationResult.Invalid(resultCode)))
        .ExchangeGoogle(new GoogleExchangeRequest("present-token"), CancellationToken.None);

    AssertProblem<ObjectResult>(result, StatusCodes.Status401Unauthorized, resultCode);
}

[Fact]
public async Task AdminExchange_ApprovedIdentityReturnsHttp200AndAdminToken()
{
    var result = await Controller(validator: new FakeGoogleValidator(GoogleAdminIdentityValidationResult.Valid("subject-1", "admin@example.test", true, "Admin User")))
        .ExchangeGoogle(new GoogleExchangeRequest("present-token"), CancellationToken.None);

    var ok = Assert.IsType<OkObjectResult>(result);
    var response = Assert.IsType<GoogleExchangeResponse>(ok.Value);
    Assert.True(response.Success);
    Assert.Equal(PlatformAppV0Contract.AdminAuthenticated, response.ResultCode);
    Assert.Equal("admin-token", response.AdministratorToken);
    Assert.NotNull(response.Administrator);
    Assert.Equal("ADMIN@EXAMPLE.TEST", response.Administrator!.NormalizedEmail);
    Assert.Equal("Admin User", response.Administrator.DisplayName);
}
```

## 12. C# before/after correction

### Before

```csharp
if (string.IsNullOrWhiteSpace(request.IdToken) || string.IsNullOrWhiteSpace(options.Value.GoogleClientId))
{
    return Unauthorized(Problem(PlatformAppV0Contract.PlatformAdminNotAuthenticated, correlationId));
}

GoogleJsonWebSignature.Payload payload;
try
{
    payload = await GoogleJsonWebSignature.ValidateAsync(request.IdToken, new GoogleJsonWebSignature.ValidationSettings
    {
        Audience = [options.Value.GoogleClientId]
    });
}
catch
{
    return Unauthorized(Problem(PlatformAppV0Contract.PlatformAdminNotAuthenticated, correlationId));
}
```

### After

```csharp
if (string.IsNullOrWhiteSpace(request.IdToken))
{
    return StatusCode(StatusCodes.Status401Unauthorized, Problem(PlatformAppV0Contract.GoogleIdTokenNotSent, correlationId));
}

if (string.IsNullOrWhiteSpace(options.Value.GoogleClientId))
{
    return StatusCode(StatusCodes.Status401Unauthorized, Problem(PlatformAppV0Contract.GoogleClientIdNotConfigured, correlationId));
}

var google = await googleIdentity.ValidateAsync(request.IdToken, options.Value.GoogleClientId, cancellationToken);
if (!google.Success)
{
    return StatusCode(StatusCodes.Status401Unauthorized, Problem(google.ResultCode, correlationId));
}
```

Branch that created the original generic 401:

```text
string.IsNullOrWhiteSpace(options.Value.GoogleClientId)
```

## 13. Exact files changed

Source files touched:

- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts\PlatformAppV0Contracts.cs`
- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Application\PlatformAppV0ApiClient.cs`
- `E:\Project2026\PlatformAppV0\tests\OBM.PlatformAppV0.Tests\Phase1ContractTests.cs`
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\PlatformAppV0Module.cs`
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs`
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Security\GoogleAdminIdentityValidator.cs`
- `E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0\PlatformAppV0Phase1Tests.cs`

Pre-existing prompt002/prompt003 files still dirty from earlier:

- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Program.cs`
- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Pages\Home.razor`
- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Routes.razor`
- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\_Imports.razor`

## 14. Exact correction

- Split generic admin exchange 401 into deterministic result codes.
- Added server-side Google token validator abstraction.
- Kept exchange endpoint `[AllowAnonymous]`.
- Added PlatformAppV0 API options fallback from canonical Google auth keys and ApiServer user-secrets.
- Added tests for missing/invalid token, invalid issuer/audience/expired, unverified email, missing allowlist, identity mismatch and approved success.
- Preserved PlatformAppV0 redirect mapping of HTTP status and API resultCode.

## 15. Diagnostics/result codes

Current runtime blocker:

```text
GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

Expected after config is supplied:

- dummy token returns `GOOGLE_ID_TOKEN_VALIDATION_FAILED` or more specific validation result;
- real Google token with mismatched audience returns `GOOGLE_ID_TOKEN_INVALID_AUDIENCE`;
- real Google token with approved identity returns `PLATFORM_ADMIN_AUTHENTICATED`.

## 16. Build commands/results

Command:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln
```

Result:

```text
PASS
0 warnings
0 errors
```

## 17. Test commands/counts

Command:

```text
dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 11
Skipped: 0
Total: 11
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 13
Skipped: 0
Total: 13
```

ApiServer build-enabled test emitted broad pre-existing warnings outside the PlatformAppV0 slice. No PlatformAppV0 focused test failed.

## 18. Process/PID/port/binary evidence

Processes stopped for build locks:

```text
PlatformAppV0 PID 18004 stopped
ApiServer01 PID 48088 stopped
```

Final restarted runtime:

```text
ApiServer01 PID 26724
Path E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe
Port 7161 listening

OBM.PlatformAppV0 PID 55268
Path E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe
Port 7012 listening
```

Runtime probes:

```text
GET http://127.0.0.1:7161/api/platform-v0/readiness
HTTP 200
{"success":true,"resultCode":"PLATFORM_V0_PHASE1_READY","implementationState":"Phase1"}

GET https://localhost:7012/
HTTP 200
```

## 19. Acceptance matrix A-G

A. Google login = PASS  
Status: not re-run by Codex; requires operator Google browser session.

B. Click Authorize Platform Administrator  
Status: not re-run by Codex; route prepared.

C. URL no longer contains `HTTP_401_PLATFORM_ADMIN_NOT_AUTHENTICATED`  
Status: source corrected; current expected blocked URL would contain `HTTP_401_GOOGLE_CLIENT_ID_NOT_CONFIGURED` until config is supplied.

D. Administrator authorization = PASS  
Status: BLOCKED by missing ApiServer Google ClientId config.

E. Approved identity populated  
Status: BLOCKED by missing ApiServer Google ClientId config.

F. Create/Select Tenant and POS1 enabled  
Status: BLOCKED until D/E pass.

G. Tenant/POS click returns success or explicit backend failure  
Status: not reached.

## 20. Remaining blockers/risks/unverified

Remaining blocker:

```text
ApiServer PlatformAppV0 Phase1 runtime needs Google ClientId and approved administrator email or subject configured.
```

Use placeholders only:

```powershell
dotnet user-secrets set "Authentication:Google:ClientId" "<GOOGLE_OAUTH_CLIENT_ID>" --project "E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj"
dotnet user-secrets set "Authentication:Google:ApprovedAdminEmail" "<APPROVED_ADMIN_EMAIL>" --project "E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj"
```

Optional subject allowlist:

```powershell
dotnet user-secrets set "Authentication:Google:ApprovedGoogleSubject" "<APPROVED_GOOGLE_SUBJECT>" --project "E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj"
```

Do not place ClientSecret in ApiServer for this exchange; the API validates the Google ID token audience and allowlist.

## 21. Source Git state/no push

Source Git status for touched/relevant files includes tracked modifications and untracked files under allowed task folders. No source commit was created and no source push was attempted.

No `git add .`, `git add -A`, reset, clean, stash, checkout or restore was used.

No WPF redeem, Pairing Code generation, PlatformEnrollment, Phase 2 or local POS database activity was performed.

## 22. Exact user retest steps

1. Configure ApiServer Google ClientId and approved admin email/subject using local user-secrets with placeholders above.
2. Restart ApiServer PlatformAppV0 Phase1 profile.
3. Restart PlatformAppV0 profile.
4. Open `https://localhost:7012`.
5. Sign out if an old browser auth state is present.
6. Click `Continue with Google`.
7. Complete Google login.
8. Click `Authorize Platform Administrator`.
9. Verify URL no longer contains `HTTP_401_PLATFORM_ADMIN_NOT_AUTHENTICATED`.
10. If still blocked, record the new `admin_authorize_result`.
11. PASS target after config:
    - `Platform administrator authorization state = PASS`
    - approved identity populated
    - `Create/Select Tenant and POS1` enabled
12. Do not create Pairing Code, start WPF redeem, or begin Phase 2 in this retest.

## 23. Coordination commit

This report should be committed only to:

```text
lequochung99/OBM-AI-Coordination
report/report004.md
```
