# Andiswa Consultation App

**POE submission · Team LogicWave · Rosebank International**

A cross-platform mobile app for **Andiswa Business Consultation**, a creative studio in Port Elizabeth that offers graphic design, branding and business consultation. The app lets the public browse services, request quotes, and stay in touch with the studio through a mobile-first experience.

---

## Team

| Name | Student number | Role |
|---|---|---|
| Ondela Skwatsha | ST10454909 | Project Manager |
| Afeziwe Thandani | ST10448454| System Analyst |
| Vuyolwethu Sicwebi |ST10453989 | Web Designer |

**Client:** Andiswa Phewa, Founder of Andiswa Business Consultation

---

## Contents

1. [How the requirements are met](#1-how-the-requirements-are-met)
2. [Features](#2-features)
3. [System design](#3-system-design)
4. [How the app, website and database work together](#4-how-the-app-website-and-database-work-together)
5. [Technologies](#5-technologies)
6. [Security design](#6-security-design)
7. [How to run the app](#7-how-the-app)
8. [How to test the requirements](#8-how-to-test-the-requirements)
9. [Project structure](#9-project-structure)
10. [Known limitations](#10-known-limitations)
11. 11.You Tube (Demonistration Video)
12. SQL Database Setup
13. Automated Test
14. [References](#11-references)

---

## 1. How the requirements are met

| Requirement | How it is implemented | Where to find it |
|---|---|---|
| **The user must be able to register and log in** | A **Create account** page (name, email, password, confirm password) and a **Log in** page. After registering, the user is logged in straight away and receives access to their dashboard. | App pages for Account, Login and Register |
| **The password must be encrypted** | Passwords are protected with **bcrypt hashing** (work factor 12, with a random salt per password). The password itself is never stored, only the hash. | API security section |
| **Make use of single sign-on (SSO)** | **Continue with Google** on both the login and register pages. The user signs in with their existing Google account; Google confirms who they are and the app receives a secure sign-in result. | Google SSO integration |

---

## 2. Features

### Navigation
A **bottom navigation bar** is shown on every main page, the same on Android and Windows:
**Home · Services · Portfolio · Contact · More · Account**

### Public pages
| Page | Purpose |
|---|---|
| **Home** | Studio introduction, key statistics, preview of services and a client testimonial |
| **Services** | Ten services; tapping one opens the Contact form with that service already selected |
| **Portfolio** | Past work with search and filters (branding, flyers, posters, social media) and a detail page per project |
| **Contact** | Quote request form, studio contact details and frequently asked questions |
| **More** | About Us, Team, Testimonials, and **Settings** (dark mode and font style) |

### Client portal (Account)
| Page | Purpose |
|---|---|
| **Log in** | Email and password, or Continue with Google |
| **Create account** | Registration with password rules shown on the form |
| **Dashboard** | *Welcome, [name]*, the client's **quote requests** with their status (for example *New* or *Accepted*), and their **projects** once a quote is accepted. Pull down to refresh. Login tokens are retained securely. |

### Offline support
The app keeps a saved copy of the studio information, so pages still load without internet. A quote request sent while offline is stored on the device and sent automatically when the connection returns.

---

## 3. System design

The app uses a **three-tier architecture**. The app never connects to the database directly; every request goes through our own Web API. The API uses the **same database as the company website**, which keeps the system consistent and secure.

```
 ┌──────────────────────┐        ┌──────────────────────┐        ┌──────────────────┐
 │  Presentation tier   │  HTTP  │  Application tier    │        │  Data tier       │
 │  .NET MAUI app       │ ─────▶ │  ASP.NET Core        │ ─────▶ │  MySQL database  │
 │  (Android, Windows)  │ ◀───── │  Web API             │ ◀───── │  (shared with    │
 │                      │  JSON  │                      │        │   the website)   │
 └──────────────────────┘        └──────────┬───────────┘        └──────────────────┘
                                             │ OAuth 2.0
                                             ▼
                                      Google (SSO only)
```

**Why an API instead of connecting the app to the database?**
- The database password stays on the server and is never inside the app.
- All rules (validation, password hashing, spam limits) are enforced in one place and cannot be skipped by changing the app.
- The website and the app share one source of data.

### API endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Checks that the API and database are working |
| GET | `/api/studio` | Studio details for the Home and Contact pages |
| POST | `/api/quotes` | Saves a quote request |
| POST | `/api/client/register` | Creates a client account (password stored as a bcrypt hash) |
| POST | `/api/client/login` | Logs in with email and password and returns a login token |
| GET | `/api/client/dashboard` | The logged-in client's quote requests and projects (token required) |
| GET | `/api/sso/google` | Starts Google sign-in in the browser |
| GET | `/api/sso/result` | The app collects the completed Google sign-in |

---

## 4. How the app, website and database work together

The mobile app, the company website and the database are **one connected system**. The app and the website both read from and write to the **same MySQL database**, so a change made in one place shows in the other.

```
   Mobile app  ──▶  Web API  ──▶  ┌─────────────────┐  ◀──  Website (PHP)
   (clients)                     │ MySQL database  │      admin page
                                 │ andiswa_business│      (studio staff)
   Mobile app  ◀──  Web API  ◀── └─────────────────┘  ──▶  client portal
```

### What updates where

| Someone does this… | …it is saved in | …and it shows up here |
|---|---|---|
| Sends a quote request from the app's **Contact** page | `quotes` table, status **New** | The website's **admin page**, marked *(Sent from the mobile app)* |
| The admin changes a quote's status on the website | `quotes` table | The client's **app dashboard** under *Your quote requests* |
| The admin creates a project for an accepted quote | `projects` table | The client's **app dashboard** under *Your projects* |
| The admin changes the studio details on the website | `settings` table | The app's **Home** and **Contact** pages |
| A client creates an account in the app | `clients` table | The client can also log in to the **website's client portal** |

### Why all three must be running

To see these updates live, the **database**, the **website** and the **API with the app** must all be running at the same time:

- The **database** (XAMPP MySQL) is where everything is stored.
- The **website** (XAMPP Apache) is where the admin sees new quotes and updates statuses and projects.
- The **API** connects the app to the database. If the API is not running, the app cannot reach the database.

After a change, **refresh the page** to see it: reload the admin page in the browser, or **pull down** on the app's dashboard or Home page.

---

## 5. Technologies

| Part | Technology |
|---|---|
| Mobile app | C#, .NET 8, .NET MAUI (Android and Windows) |
| Web API | C#, ASP.NET Core 8 Web API |
| Data access | Entity Framework Core 8 with the Pomelo MySQL provider |
| Database | MySQL (XAMPP for local testing; Azure Database for MySQL when deployed) |
| Password hashing | bcrypt (BCrypt.Net-Next) |
| Single sign-on | Google sign-in using OAuth 2.0 (Microsoft.AspNetCore.Authentication.Google) |
| API documentation and testing | Swagger UI and a Visual Studio `.http` test file |
| Hosting | Microsoft Azure App Service |
| Version control | Git and GitHub |

---

## 6. Security design

### Password protection: hashing rather than encryption
The brief asks for the password to be encrypted. We used **hashing**, which is the industry-recommended way to protect stored passwords (OWASP, n.d.):

- **Encryption is reversible.** Anyone who obtains the key can decrypt every password.
- **Hashing is one-way.** The stored value cannot be turned back into the password, even by someone with full access to the database.

We use **bcrypt**, which:
- adds a random **salt** to each password, so two users with the same password get different hashes; and
- is deliberately **slow** (work factor 12), so guessing passwords by trying millions of combinations is impractical.

At login, the API hashes the typed password with the stored salt and compares the result. The original password is never stored, logged or sent back.

Bcrypt hashes from the API use the same format as PHP's `password_hash()`, so accounts work on both the app and the website.

### Other security measures
| Measure | Detail |
|---|---|
| Password rules | At least 8 characters, including a letter and a number. Checked in the app for quick feedback and again in the API, where it cannot be bypassed. |
| Login tokens | After login the API issues a token signed with HMAC-SHA256 that expires after 30 days. A client cannot change the token to see someone else's data, because the signature would not match. |
| Secure storage on the device | The login token is stored in the device's **SecureStorage**, not in plain settings. |
| Brute-force protection | A maximum of 10 login or registration attempts per minute from one address. |
| Spam protection | A maximum of 5 quote requests per minute from one address. |
| No account discovery | A failed login always says *"Incorrect email or password"*, so the form cannot be used to find out which emails are registered. |
| SSO safety | The Google sign-in session code is random, single-use and expires after 5 minutes. The app never handles the user's Google password. |
| Secrets | The Google client secret is kept out of the source code (Visual Studio User Secrets locally, App Service settings on Azure). |

---

## 7. How to run the app

### Requirements
- Visual Studio 2022 with the **.NET Multi-platform App UI** and **ASP.NET and web development** workloads
- XAMPP (**Apache** and **MySQL**) with the `andiswa_business` database imported
- The company website's files in XAMPP's `htdocs` folder

### Steps
Run all three parts at the same time, in this order:

1. **Start the database and the website:** open the XAMPP Control Panel and start **Apache** and **MySQL**.
2. **Open the website's admin page:** in a browser, go to `http://localhost/<website-folder>/admin.php` and log in as the admin. Keep this tab open.
3. **Start the API:** open `AndiswaConsultationApp.Api.sln` and press **F5**. Swagger opens at `http://localhost:5080/swagger`. Opening `http://localhost:5080/api/health` should show a successful database response.
4. **Start the app:** open `AndiswaConsultationApp.sln` in a second Visual Studio window, choose **Windows Machine** (recommended) or an **Android Emulator**, and press **F5**.

Login credentials for admin side:
5. Username:admin 
   Password:admin@2026

The app connects to the API automatically: `localhost` on Windows, and `10.0.2.2` on the Android emulator.

### Google sign-in
Google sign-in needs a Client ID and Client secret from a Google Cloud project, with `http://localhost:5080/signin-google` as an authorised redirect URI. These are kept out of the repository for security reasons.

```json
{
  "Authentication": {
    "Google": { "ClientId": "...", "ClientSecret": "..." }
  }
}
```

Without them, the rest of the app works normally and the Google button shows *"Google sign-in is not set up yet"*. The team can provide test credentials, or demonstrate this feature, on request.

---

## 8. How to test the requirements

| # | Test | Expected result |
|---|---|---|
| 1 | Tap **Account**, then **Create an account**. Enter a name, an email and the password `abc`. | The form refuses it: *"Your password must be at least 8 characters long."* |
| 2 | Register again with the password `Password123`. | The account is created and the dashboard opens with *Welcome, [name]*. |
| 3 | In phpMyAdmin, open the `clients` table and find the new account. | The password column holds a long bcrypt hash starting with `$2`, **not** `Password123`. |
| 4 | Tap **Log out**, then log in with the same email and a wrong password. | *"Incorrect email or password."* |
| 5 | Log in with the correct password. | The dashboard opens. |
| 6 | Close the app completely and open it again, then tap **Account**. | The dashboard opens without logging in again. |
| 7 | Tap **Log out**, then **Continue with Google**, and sign in with a Google account. | The browser shows *"You're signed in"*. Back in the app, the dashboard opens within a few seconds. |
| 8 | Send a quote from **Contact** using the same email as the account, then open the dashboard and pull down to refresh. | The new quote appears under **Your quote requests** with status *New*. |
| 9 | Send a quote from the app's **Contact** page, then refresh the website's **admin page**. | The quote appears on the admin page with status *New* and the note *"(Sent from the mobile app)"*. |
| 10 | On the admin page, change that quote's status to **Accepted**. In the app, pull down on the dashboard. | The app now shows *Status: **Accepted***. |
| 11 | On the admin page, change the studio's phone number. In the app, pull down on the **Home** page and open **Contact**. | The new phone number is shown in the app. |

---

## 9. Project structure

```
AndiswaConsultationApp/                 Mobile app (.NET MAUI)
├── AppShell.xaml                       Pages and navigation routes
├── App.xaml                            Theme, styles and the bottom-bar page template
├── Controls/                           Reusable parts: bottom navigation bar, Google button, footer, error box
├── Pages/                              All screens, including the client login, register and dashboard
├── Models/                             Data classes and the website's written content
├── Services/                           API calls, navigation, login session, theme
└── Resources/                          Images, fonts, icon, splash screen, offline data

AndiswaConsultationApp.Api/             Web API (ASP.NET Core)
├── Program.cs                          Start-up: database, security, rate limits, Google sign-in
├── Controllers/                        Studio, Quotes, Client (register/login/dashboard), Sso, Health
├── Data/                               Database context and table mappings
├── Dtos/                               Data sent between the app and the API
└── Services/                           Login token creation and checking
```

The code is commented throughout to explain what each part does.

---

## 10. Known limitations

- **Google sign-in is in testing mode.** Until the Google project is verified by Google, only Google accounts added as test users can sign in.
- **Google sign-in on the Android emulator.** Google only accepts `localhost` or `https` addresses. Against the API running on a PC, Google sign-in therefore works in the Windows app; on Android it may require an emulator or trusted redirect configuration.
- **Projects are created by the studio.** Clients can see their projects, but projects are created by the admin on the website once a quote is accepted.

---
## 11.You Tube (Demonistration Video)

https://youtube.com/shorts/IRTW765c3Tw?si=_pU05Dug0TdWlYnn

## 12. SQL Database Setup

## 12

![](./gitPic/Azure.jpg)


## 13. Automated Test

The automated tests are in the website repository.

## 14. References

BcryptNet. n.d. *BCrypt.Net-Next*. [Online]. Available at: https://www.nuget.org/packages/BCrypt.Net-Next [Accessed 19 October 2026].

Google. n.d. *Using OAuth 2.0 to access Google APIs*. [Online]. Available at: https://developers.google.com/identity/protocols/oauth2 [Accessed 19 October 2026].

Microsoft. n.d. *.NET Multi-platform App UI documentation*. [Online]. Available at: https://learn.microsoft.com/en-us/dotnet/maui/ [Accessed 19 October 2026].

Microsoft. n.d. *Google external login setup in ASP.NET Core*. [Online]. Available at: https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/google-logins [Accessed 19 October 2026].

OWASP. n.d. *Password Storage Cheat Sheet*. [Online]. Available at: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html [Accessed 19 October 2026].

Pomelo Foundation. n.d. *Pomelo.EntityFrameworkCore.MySql*. [Online]. Available at: https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql [Accessed 19 October 2026].

