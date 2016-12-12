


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
  
    <system.webServer>
    <modules>
      <!--<remove name="FormsAuthentication" /> -->
    </modules>
  </system.webServer>
```
Replace the ``` <system.web> ``` and ``` <system.webServer> ``` tag with the above code in the web.config

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



# Database Connection Stuff

## Connection Code

##### Web.config

```cs
<connectionStrings>
    <add name="ISProject2context" connectionString="Data Source=BRIANDELROSE8C1\BRIANWORK;Initial Catalog=ISProject2;Integrated Security=True;Pooling=False"
      providerName="System.Data.SqlClient" />
  </connectionStrings>

```
Add the connection string code information:
<ol>
<li>Go to the serve explorer window on Visual Studio.</li>
<li>Expand the data connections, and double click the database that you want to connect.</li>
<li>In the window that opened, copy the connection string into the above code.</li>
<li>Name the connection what you like, but know that you will reference this connection later in the code of the project.</li>
</ol>



##### Models

Create a model:
<ol>
<li>In the folder "Models" create a new ".cs" file.</li>
<li>Copy this code, but replace the <b>REPLACE THIS</b> parts with project specific code:</li>
</ol>
```cs
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using System.Linq;
using System.Web;

namespace REPLACE THIS WITH PROJECT NAME.Models
{
    [Table("REPLACE THIS WITH THE TABLE YOU WOULD LIKE THIS MODEL TO DRAW DATA FROM")]
    public class REPLACE THIS WITH THE NAME OF THE FILE
    {
        //replace this info with all the columns from the table, COPY THE NAMES EXACTLY HOW WRITTEN ON THE TABLE IN THE DATABASE
        [Key]
        public int degID { get; set; }
        public String degName { get; set; }
        public String degCoord { get; set; }
        public String degCoordtitle { get; set; }
        public String degOfficeaddress { get; set; }
        public String degPhdEd { get; set; }
        public String degMastersEd { get; set; }
        public String degBachelorsEd { get; set; }
        public String degCoordpic { get; set; }

    }
}
```


##### DAL

Create the connection file:
<ol>
<li>Create a folder in the project called "DAL".</li>
<li>Create a ".cs" file with the <b>name of the connection string</b> in the Web.config file.</li>
<li>Copy this code, but replace the <b>REPLACE THIS</b> parts with project specific code:</li>
</ol>
```cs
using REPLACE THIS WITH PROJECT NAME.Models;
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Web;

namespace REPLACE THIS WITH PROJECT NAME.DAL
{
    public class REPLACE THIS WITH FILE NAME : DbContext
    {
        public REPLACE THIS WITH FILE NAME()
            : base("REPLACE THIS WITH FILE NAME")
        {

        }

        public DbSet<REPLACE THIS WITH MODEL NAME> REPLACE THIS WITH WHATEVER YOU WANT TO CALL IT { get; set; }
        //repeat the above code for each model
    }
}
```

##### Add to controllers specific code

Add to the controller that you want to use the database with, the following code:

Top
```cs
using System.Web.Mvc;
using blankproject.DAL;
using blankproject.Models;
using System.Data.Entity;
using System.Data;
```

Public class
```cs
public class UsersModelsController : Controller
    {
        private ISProject2context db = new ISProject2context();
```


##### Model Validations (RegEx)

```cs
//Phone Number
[RegularExpression(@"^[0-9]{0,15}$", ErrorMessage = "PhoneNumber should contain only numbers")]


//Email address
[Email(ErrorMessage = "Bad email")]


//Zip code (matches 12345 and 12345-5432)
[RegularExpression(@"^\d{5}([\-]\d{4})?$", ErrorMessage = "Invalid Zip Code")]
```


###### Notes
