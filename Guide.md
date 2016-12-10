# Login

## Username and Password

##### Step 1

###### Notes


# Login

## Username and Password

##### Logon
```cs
        // load login page to enter information
        public ActionResult Login()
        {
            return View();
        }


        //receive login credentials from login page, validating that the information is in the database and creating a session
        [HttpPost]
        public ActionResult Login(FormCollection form, bool rememberMe = true)
        {
            //load form input into variables
            String email = form["Email"].ToString();
            String password = form["Password"].ToString();


            //return whether the record with that email exists
            bool userexists = db.users.Any(m => m.userEmail.Equals(email));

            //if the user exists then continue to check password match
            if (userexists)
            {
                //load the record of that user 
                UsersModels User = db.users.SingleOrDefault(user => user.userEmail == email);

                //load up password from record into variable
                string matchingpw = User.userPassword.ToString();
                int myuserid = User.userID;
                Session["WHATEVER"] = myuserid;

                //check if password matches record
                if (string.Equals(password, matchingpw))
                {
                    FormsAuthentication.SetAuthCookie(email, rememberMe);
                    ViewBag.userid = myuserid;
                    return RedirectToAction("Degrees", "Home");

                }
                else
                    //email exists but pw doesn't match
                {
                    //if the password doesn't match then return them to the view
                    return View();

                    //return error message
                }
            }
            else
            {
                //if neither matches then return them to the view
                return View();

                //return error message
            }

           
        }
```
Put this in a controller (usually home controller), wherever the login view is stored.


##### Logoff
```cs
//Logging out
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult LogOff()
        {
            //delete session
            Session.Abandon();
            System.Web.Security.FormsAuthentication.SignOut();
            return RedirectToAction("Index", "Home");
        }
```
Put this in a controller (usually home controller).

##### Additional required modifications in other files
###### Web.config
```cs
<system.web>
    <authentication mode="Forms">
      <forms loginUrl="~/Home/Login" timeout="2880" />
    </authentication>
    
    <compilation debug="true" targetFramework="4.5.1" />
    <httpRuntime targetFramework="4.5.1" />
  </system.web>
```
Replace the ``` <system.web> ``` tag with the above code in the web.config

###### Startup.Auth
```cs
public void ConfigureAuth(IAppBuilder app)
        {
            /*
            // Configure the db context, user manager and signin manager to use a single instance per request
            app.CreatePerOwinContext(ApplicationDbContext.Create);
            app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
            app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);

            // Enable the application to use a cookie to store information for the signed in user
            // and to use a cookie to temporarily store information about a user logging in with a third party login provider
            // Configure the sign in cookie
            app.UseCookieAuthentication(new CookieAuthenticationOptions
            {
                AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
                LoginPath = new PathString("/Home/Login"),
                Provider = new CookieAuthenticationProvider
                {
                    // Enables the application to validate the security stamp when the user logs in.
                    // This is a security feature which is used when you change a password or add an external login to your account.  
                    OnValidateIdentity = SecurityStampValidator.OnValidateIdentity<ApplicationUserManager, ApplicationUser>(
                        validateInterval: TimeSpan.FromMinutes(30),
                        regenerateIdentity: (manager, user) => user.GenerateUserIdentityAsync(manager))
                }
            });            
            app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);

            // Enables the application to temporarily store user information when they are verifying the second factor in the two-factor authentication process.
            app.UseTwoFactorSignInCookie(DefaultAuthenticationTypes.TwoFactorCookie, TimeSpan.FromMinutes(5));

            // Enables the application to remember the second login verification factor such as phone or email.
            // Once you check this option, your second step of verification during the login process will be remembered on the device where you logged in from.
            // This is similar to the RememberMe option when you log in.
            app.UseTwoFactorRememberBrowserCookie(DefaultAuthenticationTypes.TwoFactorRememberBrowserCookie);

            // Uncomment the following lines to enable logging in with third party login providers
            //app.UseMicrosoftAccountAuthentication(
            //    clientId: "",
            //    clientSecret: "");

            //app.UseTwitterAuthentication(
            //   consumerKey: "",
            //   consumerSecret: "");

            //app.UseFacebookAuthentication(
            //   appId: "",
            //   appSecret: "");

            //app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
            //{
            //    ClientId = "",
            //    ClientSecret = ""
            //});*/
        }
```

###### Authorizing
Add an ``` [Authorize] ``` at the top of the ActionResult method to show a view.




