# Blazor Server + Authentication Starter

**A production-ready Blazor Server template with ASP.NET Identity authentication and PostgreSQL.**

Build secure web applications with built-in user registration, login, and role-based access control. No configuration required - just deploy and start building authenticated features.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/6fzPQH?referralCode=Ce0gB7&utm_medium=integration&utm_source=template&utm_campaign=generic)

## ✨ Features

- 🔐 **ASP.NET Identity** - Complete authentication system built-in
- 👤 **User Registration** - Sign up with email and password
- 🔑 **Secure Login/Logout** - Password hashing with bcrypt
- 🛡️ **Protected Pages** - Require authentication for sensitive routes
- 👥 **User Management** - Built-in account management pages
- 🐘 **PostgreSQL** - Production-ready database for user storage
- ⚡ **Blazor Server** - Interactive C# components with real-time updates
- 🎨 **Clean UI** - Professional authentication pages included
- 🐳 **Docker Optimized** - Multi-stage builds for production
- 🚂 **Railway Ready** - Zero-config deployment with auto-migration

## 🚀 Quick Start

### Deploy to Railway

Click the "Deploy on Railway" button above. Railway will automatically:
- Build your Blazor application using Docker
- Provision a PostgreSQL database
- Run Identity migrations (creates user tables)
- Connect everything together
- Generate a public URL with SSL

**Then visit:** `https://your-app.railway.app/Account/Register`

### Local Development

**Prerequisites:**
- .NET 9 SDK
- PostgreSQL (or use Docker)

**Steps:**
```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/blazor-auth-starter.git
cd blazor-auth-starter

# Set up PostgreSQL (or use Docker)
docker run --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres

# Update appsettings.json with your connection string
# (Already configured for localhost:5432)

# Run migrations
dotnet ef database update

# Run the application
dotnet run

# Open browser to https://localhost:5001
```

## 📁 Project Structure
```
blazor-auth-starter/
├── Components/
│   ├── Account/              # Authentication pages
│   │   ├── Pages/           # Login, Register, Manage
│   │   └── Shared/          # Account layout components
│   ├── Layout/              # App layout components
│   ├── Pages/               # Application pages
│   │   ├── Home.razor       # Public home page
│   │   ├── Auth.razor       # Example: Requires login
│   │   └── Weather.razor    # Protected weather page
│   └── App.razor            # Root component
├── Data/
│   ├── ApplicationDbContext.cs    # EF Core Identity context
│   └── ApplicationUser.cs         # User model
├── Migrations/              # Database migrations
├── Dockerfile               # Multi-stage Docker build
├── railway.toml             # Railway configuration
├── Program.cs               # Application configuration
└── README.md                # Documentation
```

## 🔐 Authentication Features

### Built-In Pages

**Account Management:**
- `/Account/Register` - User registration
- `/Account/Login` - User login
- `/Account/Logout` - Logout (POST endpoint)
- `/Account/Manage` - User profile management
- `/Account/Manage/Email` - Change email
- `/Account/Manage/Password` - Change password
- `/Account/Manage/TwoFactorAuthentication` - 2FA setup

### Protecting Pages

Use the `@attribute [Authorize]` directive:
```razor
@page "/dashboard"
@attribute [Authorize]

<PageTitle>Dashboard</PageTitle>

<h1>Dashboard</h1>
<p>This page requires authentication!</p>

<AuthorizeView>
    <Authorized>
        <p>Hello, @context.User.Identity?.Name!</p>
    </Authorized>
</AuthorizeView>
```

### Role-Based Authorization
```razor
@attribute [Authorize(Roles = "Admin")]

<h1>Admin Only Page</h1>
```

### Check Authentication in Code
```razor
@inject AuthenticationStateProvider AuthenticationStateProvider

@code {
    private string? userName;

    protected override async Task OnInitializedAsync()
    {
        var authState = await AuthenticationStateProvider.GetAuthenticationStateAsync();
        var user = authState.User;

        if (user.Identity?.IsAuthenticated ?? false)
        {
            userName = user.Identity.Name;
        }
    }
}
```

## 🛠️ Customization

### Add Custom User Properties

**Update** `Data/ApplicationUser.cs`:
```csharp
public class ApplicationUser : IdentityUser
{
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

**Create migration:**
```bash
dotnet ef migrations add AddUserProperties
```

**Deploy - Railway runs migration automatically!**

### Add Roles

**In** `Program.cs`, after building the app:
```csharp
using (var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
    
    if (!await roleManager.RoleExistsAsync("Admin"))
    {
        await roleManager.CreateAsync(new IdentityRole("Admin"));
    }
}
```

**Assign role to user:**
```csharp
var userManager = scope.ServiceProvider.GetRequiredService<UserManager<ApplicationUser>>();
var user = await userManager.FindByEmailAsync("admin@example.com");
await userManager.AddToRoleAsync(user, "Admin");
```

### Email Confirmation (Optional)

By default, email confirmation is disabled for easier development. To enable:

1. **Update** `Program.cs`:
```csharp
builder.Services.AddIdentityCore<ApplicationUser>(options => 
    options.SignIn.RequireConfirmedAccount = true) // Already set to true
```

2. **Implement email sender** - Replace `IdentityNoOpEmailSender`:
```csharp
builder.Services.AddTransient<IEmailSender<ApplicationUser>, YourEmailService>();
```

### External Authentication (Google, Microsoft, etc.)

Add external providers in `Program.cs`:
```csharp
builder.Services.AddAuthentication()
    .AddGoogle(options =>
    {
        options.ClientId = builder.Configuration["Google:ClientId"];
        options.ClientSecret = builder.Configuration["Google:ClientSecret"];
    });
```

## 🔒 Security Features

### Included by Default

✅ **Password Hashing** - Uses ASP.NET Identity's secure hashing (bcrypt-based)  
✅ **CSRF Protection** - Anti-forgery tokens on all forms  
✅ **XSS Protection** - Blazor automatically escapes HTML  
✅ **SQL Injection Protection** - Entity Framework parameterized queries  
✅ **HTTPS** - Enforced in production, automatic SSL on Railway  
✅ **Secure Cookies** - HttpOnly, Secure, SameSite configured  

### Password Requirements

Default requirements (configurable in `Program.cs`):
- Minimum 6 characters
- At least 1 uppercase letter
- At least 1 lowercase letter
- At least 1 digit
- At least 1 non-alphanumeric character

**Customize:**
```csharp
builder.Services.AddIdentityCore<ApplicationUser>(options => 
{
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireLowercase = true;
})
```

## 📊 Database Schema

ASP.NET Identity creates these tables automatically:

- `AspNetUsers` - User accounts
- `AspNetRoles` - Roles (Admin, User, etc.)
- `AspNetUserRoles` - User-role mappings
- `AspNetUserClaims` - Custom user claims
- `AspNetUserLogins` - External login providers
- `AspNetUserTokens` - Authentication tokens
- `AspNetRoleClaims` - Role-based claims

## 🎓 Common Use Cases

### Perfect For:

📊 **Internal Dashboards** - Employee-only admin panels  
💼 **SaaS Applications** - Multi-tenant user management  
🛒 **E-Commerce** - Customer accounts and order history  
📝 **Content Management** - Author/editor role-based access  
🏢 **Business Applications** - CRM, inventory, invoicing with user access  
👥 **Social Platforms** - User profiles, followers, content creation  

## ⚙️ Environment Variables

Railway automatically sets:
- `DATABASE_URL` - PostgreSQL connection (auto-configured)
- `PORT` - Application port
- `ASPNETCORE_ENVIRONMENT` - Set to `Production`

**Optional variables:**
- `EmailSender__ApiKey` - For email service
- `Google__ClientId` - For Google OAuth
- `Google__ClientSecret` - For Google OAuth

## 🚀 Deployment

### Railway (Recommended)

1. Click "Deploy on Railway" button
2. Railway handles everything automatically
3. Visit your generated URL
4. Register your first user!

### Manual Deployment
```bash
# Build
dotnet publish -c Release

# Run migrations
dotnet ef database update

# Start app
dotnet blazor-auth-starter.dll
```

## 📚 Learn More

### ASP.NET Identity
- [Identity Documentation](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity) - Official Microsoft Identity docs
- [Identity UI Customization](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/customize-identity-model) - Customize user model and UI
- [Two-Factor Authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/2fa) - Add 2FA to your app

### Blazor Resources
- [Blazor Documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/) - Official Blazor guides
- [Blazor Security](https://learn.microsoft.com/en-us/aspnet/core/blazor/security/) - Authentication and authorization patterns

### Deployment
- [Railway Docs](https://docs.railway.app/) - Platform documentation
- [PostgreSQL on Railway](https://docs.railway.app/databases/postgresql) - Database management

## 🤝 Contributing

Contributions welcome! Submit a Pull Request.

## 📄 License

MIT License - see LICENSE file for details

---

**Built with ❤️ for the Railway community** 🚂

**Need database + auth?** This template is your perfect starting point!

**Questions?** Open an issue on GitHub or reach out on Railway Discord.