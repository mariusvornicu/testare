# forFeea
  This is where we keep track with the latest changes related to the course


### **Session 1**

We have discussed about automation:
  1. What is it
  2. Why is important
  3. Types of automation testing

The presentation will be sent via email at the end of this course.

### **Session 2:  Let's write our first UI automation tests**

**Scope:** This session scope was to create a unit test project, install all the dependencies and write some simple tests

How to create a unit test project:
  1. Open Visual studio
  2. Click File > New > Project
  3. Search for unit test: Unit Test Project(.NET Framework)
  4. Add project Name
  5. Click OK button
  
At this moment, we have created a unit test project and should have a default test class: UnitTest1. 
The class has the following particularities: 
1. It has [TestClass] annotation that identifies the class to be a test one. Without this annotation, the test under this class cannot     be recognized and there for cannot run the tests within it.
2. The class contains a test method that has a [TestMethod] annotation. This help the method to be recognized as an test method and run it accordingly.
  
Find more info on: https://docs.microsoft.com/en-us/visualstudio/test/walkthrough-creating-and-running-unit-tests-for-managed-code?view=vs-2019
  
Let's import the needed packages in order to open a chrome browser using our test. Do a right click on the created project and 
click Manage Nuget Packages. On the opened window, select Browse tab and search for Selenium.Webdriver and install it in your project. Then search for Selenium.Webdriver.ChromeDriver and install the latest version(avoid the ones that are in betha). 

Be aware that if the local installed Chrome is not the same with the installed package, it will trigger a compatibility error when running the test.
  
Let's write some code :) 
  
**REMEMBER: THE TEST DOES WHAT YOU TELL IT TO DO.**
Thats why, selenium manipulates the elements in DOM as a human would do. For now, we will use click and sendkeys.
  
Our first automated test case will try to login into application. For this, we will need a write code for the next steps:
  1. Open the browser
  2. Maximize the page
  3. Open the application URL
  4. Click Sign in button
  5. Fill user email
  6. Fill user password
  7. Click Sign in button 
  8. Assert that the current user label contains our email
    
Open the UnitTest1 class and let instantiate our driver to open the Chrome browser.

```csharp  
      var driver = new ChromeDriver(); //open chrome browser
      driver.Manage().Window.Maximize(); //maximize the window
      driver.Navigate().GoToUrl("OUR URL"); //access the SUT(System Under Test) url. In our case http://a.testaddressbook.com/
```

Until now, we have covered the first 3 steps of our test. Let's complete our test:

```csharp  
      driver.FindElement(By.Id("sign-in")).Click();
      Thread.Sleep(1000);
      driver.FindElement(By.Id("session_email")).SendKeys("email that was used for creating the account");
      driver.FindElement(By.Name("session[password]")).SendKeys("password used to create the account");
      driver.FindElement(By.CssSelector("input[value='Sign in']")).Click();
      Thread.Sleep(2000);
      Assert.AreEqual("asd@asd.asd", driver.FindElement(By.XPath("//span[@data-test='current-user']")).Text);
```

Clarification :)
  1. WebElement represents a DOM element. WebElements can be found by searching from the document root using a WebDriver instance. WebDriver API provides built-in methods to find the WebElements which are based on different properties like ID, Name, Class, XPath, CSS Selectors, link Text, etc.
  2. There are some ways of optimizing our selectors used to identify the elements in page.  
      a. For the CssSelector, the simplest way is to use the following format: tagname[attribute='attributeValue'].  
      b. For the XPath, the simplest way is to use the following format: //tagname[@attribute='attributeValue'].  
    Let's take for example the Sign in button:
    
```csharp    
    <input type="submit" name="commit" value="Sign in" class="btn btn-primary" data-test="submit" data-disable-with="Sign in">
```

    Explained: 
```csharp    
    <input(this is the tagname) 
        type(this is the attribute)="submit"(this is the value) --> The CssSelector would be: input[type='submit']
        name(this is the attribute)="commit"(this is the value) --> The XPath would be: //input[@name='commit']
        value(this is the attribute)="Sign in"(this is the value) --> The CssSelector would be: input[value='Sign in']
        class(this is the attribute)="btn btn-primary"(this is the value) --> The XPath would be: //input[@class='btn btn-primary']
        data-test(this is the attribute)="submit"(this is the value) --> The CssSelector would be: input[data-test='submit']
        data-disable-with(this is the attribute)="Sign in"(this is the value) --> The Xpath would be: //input[data-disabled-with='Sign in']
    >
```   
    
### **Session 3: Let's refine/refactor our code**

**Scope:** This session scope was to use MSTest methods to initialize/clean up our test data and to get rid of our duplicate code

One of a test case component is the prerequisite: conditions that must be met before the test case can be run.
Our code test login scenarios and we need to see what are the prerequisites.
We have identified the following steps that need to be execute before running the test:

```csharp
            var driver = new ChromeDriver(); // open the browser
            driver.Manage().Window.Maximize(); // maximize the window 
            driver.Navigate().GoToUrl("http://a.testaddressbook.com/"); // access the SUT url
            driver.FindElement(By.Id("sign-in")).Click(); // click on sign in button in order to be redirected to the login page
            Thread.Sleep(1000); // THIS sleep is a bad practice AND NEEDS TO BE DELETED AND WE WILL BURN IT WITH FIRE IN THE NEAR FUTURE
```

Also, after the test has finished running, we need to clean up the operations that we made in our test in order to not impact further test. Remember, each test is independent and should not influence the result of other tests. In our test, the clean up would mean to close the browser.

```csharp
            driver.Quit();
```

MSTest provides a way to declare methods to be called by the test runner before or after running a test.

```csharp
            [TestInitialize]
            public void TestInitialize()
            {
            }

            [TestCleanup]
            public void TestCleanup()
            {
            }
```

The method decorated by [TestInitialize] is called before running each test of the class. The method decorated by [TestCleanup] is called after running each test of the class.

First, we need to remove the init/clean up steps and to move it the according method. At this point, our tests should look like this: 

```csharp
          namespace UnitTestProject1
          {
              [TestClass]
              public class LoginTests
              {
                  //declare IWebDriver instance variable
                  //use it outside of any methods so that we can use it in various methods
                  private IWebDriver driver;

                  [TestInitialize]
                  public void SetUp()
                  {
                      //initialize the needed driver. In our case is ChromeDriver
                      driver = new ChromeDriver();
                      loginPage = new LoginPage(driver);
                      driver.Manage().Window.Maximize();
                      driver.Navigate().GoToUrl("http://a.testaddressbook.com/");
                      driver.FindElement(By.Id("sign-in")).Click();
                      Thread.Sleep(1000);
                  }

                  [TestMethod]
                  public void Login_CorrectEmail_CorrectPassword()
                  {
                      driver.FindElement(By.Id("session_email")).SendKeys("test@test.test");
                      driver.FindElement(By.Id("session_password")).SendKeys("test");
                      driver.FindElement(By.Name("commit")).Click();

                      var expectedResult = "test@test.test";
                      var actualResults = driver.FindElement(By.CssSelector("span[data-test='current-user']")).Text;

                      Assert.AreEqual(expectedResult, actualResults);
                  }

                  [TestMethod]
                  public void Login_IncorrectEmail_IncorrectPassword()
                  {
                      driver.FindElement(By.Id("session_email")).SendKeys("wrong@wrong.wrong");
                      driver.FindElement(By.Id("session_password")).SendKeys("wrong");
                      driver.FindElement(By.Name("commit")).Click();

                      var expectedResult = "Bad email or password.";
                      var actualResults = driver.FindElement(By.XPath("//div[starts-with(@class, 'alert')]")).Text;

                      Assert.AreEqual(expectedResult, actualResults);
                  } 

                  [TestCleanup]
                  public void CleanUp()
                  {
                      driver.Quit();
                  }
              }
          }   
```

ONE OF THE COMMON MISTAKE IS TO DECLARE THE IWebDriver INSTANCE VARIABLE WITHIN THE TEST INIT:

```csharp
        private IWebDriver driver;

        [TestInitialize]
        public void SetUp()
        {
            var driver = new ChromeDriver();
          //this is a method local variable and cannot be used in other methods outside init. The tests will throw null refrence since it doesn't know about the ChromeDriver instance
        }
```

Our code starts to look cleaner :)

But wait, there is more work to do. Let's say that the login page layout needs to be changed. Our test will fail after this changes.
We have only two tests and will be easy to fix it. But imagine that we have 25 login tests. Is not so funny to update all the tests.

A better approach to script maintenance is to create a separate class file which would find web elements, fill them or verify them. This class can be reused in all the scripts using that element. In future, if there is a change in the web element, we need to make the change in just 1 class file and not 10 different scripts.

This approach is called Page Object Model(POM). It helps make the code more readable, maintainable, and reusable.

Page Object model is an object design pattern in Selenium, where web pages are represented as classes, and the various elements on the page are defined as variables on the class. All possible user interactions can then be implemented as methods on the class.

Let's create the the page object that contains the elements for the login page: LoginPage.cs

Right click on the project > Add > Folder and name it PageObjects

Right click on the PageObjects folder > Add > New Item... > Add a class with name: LoginPage.cs

We need to add the objects that we use in our script in this class: email input, password input, sign in button and create a method to login the user.

There are multiple ways to declare the page objects:

First: create a method to return an IWebelement

```csharp
      private IWebelement TxtEmail()
      {
        return driver.FindElement(By.Id("session_email"));
      }
```

Second: create an IWebElement property 

```csharp
      private IWebElement TxtEmail => driver.FindElement(By.Id("session_email"));
```

Third: Extract the By separately 

```csharp
      By email = By.Id("session_email")
      private IWebElement TxtEmail => driver.FindElement(email);
```

Our login page will look like this in the end:

```csharp
      public class LoginPage
      {
          //declare the driver
          private IWebDriver driver;

          //instantiate the driver
          public LoginPage(IWebDriver browser)
          {
              driver = browser;
          }

          //create our elements
          private IWebElement Username()
          {
            return driver.FindElement(By.Id("session_email"));
          }

          private IWebElement Password()
          {
              return driver.FindElement(By.Id("session_password"));
          }

          private IWebElement LoginClick()
          {
              return driver.FindElement(By.Name("commit"));
          }

          public void LoginApplication(string username, string password)
          {
              Username.SendKeys(username);
              Password.SendKeys(password);
              LoginClick.Click();
          }
      }
```

At this point, we need to update our tests:

```csharp
    [TestClass]
    public class LoginTests
    {
        private IWebDriver driver;
        private LoginPage loginPage;

        [TestInitialize]
        public void SetUp()
        {
            driver = new ChromeDriver();
            loginPage = new LoginPage(driver);
            driver.Manage().Window.Maximize();
            driver.Navigate().GoToUrl("http://a.testaddressbook.com/");
            driver.FindElement(By.Id("sign-in")).Click();
            Thread.Sleep(1000);
        }

        [TestMethod]
        public void Login_CorrectEmail_CorrectPassword()
        {
            loginPage.LoginApplication("test@test.test", "test");

            var expectedResult = "test@test.test";
            var actualResults = driver.FindElement(By.CssSelector("span[data-test='current-user']")).Text;

            Assert.AreEqual(expectedResult, actualResults);
        }

        [TestMethod]
        public void Login_IncorrectEmail_IncorrectPassword()
        {
            loginPage.LoginApplication("weor@hdsh.asdhg", "asd");

            var expectedResult = "Bad email or password.";
            var actualResults = driver.FindElement(By.XPath("//div[starts-with(@class, 'alert')]")).Text;

            Assert.AreEqual(expectedResult, actualResults);
        }


        [TestCleanup]
        public void CleanUp()
        {
            driver.Quit();
        }
    }
```

We still have work to do. But chill out and see you at the next seminar.

### **Session 4: Let's write an UI automation tests for adding a new address**

**Scope:** This session scope was to create a unit test project for adding addresses and to keep the code clean

In order to add a new address, we will need a write code for the next steps:
1.	Open the browser
2.	Maximize the page
3.	Open the application URL
4.	Click Sign in button (NavigateToLoginPage method)
5.	Fill user email and password, then click Sign in button (LoginApplication method)
6.	Navigate to addresses page
7.	Navigate to add address page
8.	Complete the form with mandatory fields and click Save button
9.	Assert that the success message is shown

For step 4 we can create a method in page object LoginPage.cs and use it in Setup():

```csharp
    public void NavigateToLoginPage()
    {
        BtnSignIn().Click();
    }
```

At step 5, another method can contain login actions (fill user email and password, click Sign in button) and should be added in page object for LoginPage.cs:

```csharp
    public void LoginApplication(string username, string password)
    {
        TxtUsername().SendKeys(username);
        TxtPassword().SendKeys(password);
        BtnLogin().Click();
    }
```

In order to navigate to addresses page, we need to create a page object HomePage.cs that contains elements and method for this page:

```csharp
    public class HomePage
    {
        private IWebDriver driver;

        public HomePage(IWebDriver browser)
        {
            driver = browser;
        }

        private IWebElement LblUserEmail => driver.FindElement(By.CssSelector("span[data-test='current-user']"));

        private IWebElement BtnAddresses => driver.FindElement(By.CssSelector("[data-test='addresses']"));

        public string UserEmailText => LblUserEmail.Text;

        public AddressesPage NavigateToAddressesPage()
        {
            BtnAddresses.Click();
            return new AddressesPage(driver);
        }
    }
```

Next step is to navigate to add address page. For this, we need another page object AddressesPage.cs which contains New Address button declaration and method to clicks on the element:

```csharp
    public class AddressesPage
    {
        private IWebDriver driver;

        public AddressesPage(IWebDriver browser)
        {
            driver = browser;
        }

        private IWebElement BtnNewAddress => driver.FindElement(By.CssSelector("[data-test='create']"));


        public AddAdressPage NavigateToAddAddressPage()
        {
            BtnNewAddress.Click();
            return new AddAdressPage(driver);
        }
    }
```

For step 8 (Complete the form with mandatory fields and click Save button), we will create a page object that contains the elements for the add address page: AddAdressPage.cs.
We need to add the objects that we use in our script in this class: first name, last name, address, city, zip code, birthday and color inputs, state dropdown, country select, save button and create a method to add the address.
Our add address page will look like this:

```csharp
    public class AddAdressPage
    {
        private IWebDriver driver;

        public AddAdressPage(IWebDriver browser)
        {
            driver = browser;
        }

        private IWebElement TxtFirstName => driver.FindElement(By.Id("address_first_name"));

        private IWebElement TxtLastName => driver.FindElement(By.Id("address_last_name"));

        private IWebElement TxtAddress1 => driver.FindElement(By.Id("address_street_address"));

        private IWebElement TxtCity => driver.FindElement(By.Id("address_city"));

        private IWebElement DdlState => driver.FindElement(By.Id("address_state"));

        private IWebElement TxtZipCode => driver.FindElement(By.Id("address_zip_code"));

        private IList<IWebElement> LstCountry => driver.FindElements(By.CssSelector("input[type=radio]"));

        private IWebElement TxtBirthday => driver.FindElement(By.Id("address_birthday"));

        private IWebElement ClColor => driver.FindElement(By.Id("address_color"));

        private IWebElement BtnSave => driver.FindElement(By.Name("commit"));

        public void AddAddress()
        {
            TxtFirstName.SendKeys("test");
            TxtLastName.SendKeys("test");
            TxtAddress1.SendKeys("test");
            TxtCity.SendKeys("test");
            var selectState = new SelectElement(DdlState);
            selectState.SelectByText("Hawaii");
            TxtZipCode.SendKeys("test");
            LstCountry[1].Click();

            var js = (IJavaScriptExecutor) driver;
            js.ExecuteScript("arguments[0].setAttribute('value', arguments[1])", ClColor, "#FF0000");
            BtnSave.Click();
            Thread.Sleep(2000);
        }
     }
```

In order to finish the automated tests, we need to make an assertion. For this test, will be the success message shown in the address details page, after saving it. Another page object will be created: AddressDetailsPage.cs

```csharp
    public class AddressDetailsPage
    {
        private IWebDriver driver;

        public AddressDetailsPage(IWebDriver browser)
        {
            driver = browser;
        }

        public IWebElement LblSuccess => driver.FindElement(By.CssSelector("[data-test='notice']"));
    }
```

At this point, our automated tests will look like this:

```csharp
    [TestClass]
    public class AddAddressTests
    {
        private IWebDriver driver;
        private AddAdressPage addAddressPage;

        [TestInitialize]
        public void Setup()
        {
            driver = new ChromeDriver();
            var loginPage = new LoginPage(driver);
            driver.Manage().Window.Maximize();
            driver.Navigate().GoToUrl("http://a.testaddressbook.com/");
            loginPage.NavigateToLoginPage();
            Thread.Sleep(1000);
            loginPage.LoginApplication("test@test.test", "test");

            var homePage = new HomePage(driver);
            Thread.Sleep(1000);
            var addressesPage = homePage.NavigateToAddressesPage();
            Thread.Sleep(1000);
            addAddressPage = addressesPage.NavigateToAddAddressPage();
            Thread.Sleep(1000);
        }

        [TestMethod]
        public void Go_To_AddAddressPage()
        {
            addAddressPage.AddAddress();
            var addressDetails = new AddressDetailsPage(driver);
            var message = "Address was successfully created.";
            Assert.AreEqual(message, addressDetails.LblSuccess.Text);
        }


        [TestCleanup]
        public void CleanUp()
        {
            driver.Quit();
        }
    }
```

### **Session 5: Let’s continue to refactor our project**

**Scope:** This session scope was to refactored our code and replace Thread.Sleep with efficient waits

In order to fill in Birthday input, we need to respect the format data. In this case is dd-MMM-yyyy:

```csharp
            TxtBirthday.SendKeys(address.TxtBirthdayDay);
```

Another thing we need to change is the way we navigate from adding address page to address details page. For this, we need to create a method in page object AddAdressPage.cs:

```csharp
        public AddressDetailsPage NavigateToAddressDetailsPage()
        {
            return new AddressDetailsPage(driver);
        }
```

Then, called it in the AddAddressTests.cs:

```csharp
            var addressDetails = addAddressPage.NavigateToAddressDetailsPage();
```

To parametrize AddAddress method in an efficient way, we can create a business object class called AddAddressBO.cs which will contain the objects needed in the process of adding an address: 

```csharp
    public class AddAddressBO
    {
        public string TxtFirstName = "test";
        public string TxtLastName = "test";
        public string TxtAddress1 = "test";
        public string TxtCity = "test";
        public string TxtState = "Hawaii";
        public string TxtZipCode = "test";
        public string TxtBirthdayDay = "05";
        public string TxtBirthdayMonth = "03";
        public string TxtBirthdayYear = "2000";
        public string TxtColor = "#FF0000";
    }
```

Then, use it in the AddAddress method as a parameter and to access his properties:

```csharp
        public void AddAddress(AddAddressBO address)
        {
            TxtFirstName.SendKeys(address.TxtFirstName);
            TxtLastName.SendKeys(address.TxtLastName);
              ... and so on
        }
```

Now, we can move on to the **Wait Strategy** and how to use it.

There are explicit and implicit waits in Selenium Web Driver. Waiting is having the automated task execution elapse a certain amount of time before continuing with the next step. 

You should choose to use Explicit or Implicit Waits.

**•	Thread.Sleep**

In particular, this pattern of sleep is an example of explicit waits. So this isn’t actually a feature of Selenium WebDriver, it’s a common feature in most programming languages though.

Thread.Sleep() does exactly what you think it does, it sleeps the thread.

Example:

```csharp
            Thread.Sleep(2000);
```

Warning! Using Thread.Sleep() can leave to random failures (server is sometimes slow), you don't have full control of the test and the test could take longer than it should. It is a good practice to use other types of waits.

**•	Implicit Wait**

WebDriver will poll the DOM for a certain amount of time when trying to find an element or elements if they are not immediately available

Example:

```csharp
            driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(1);
```

**•	Explicit Wait** - Wait for a certain condition to occur before proceeding further in the code

In practice, we recommend that you use Web Driver Wait in combination with methods of the Expected Conditions class that reduce the wait. If the element appeared earlier than the time specified during Web Driver wait initialization, Selenium will not wait but will continue the test execution.

Example 1:

```csharp
            var wait = new WebDriverWait(driver, TimeSpan.FromSeconds(2));
            wait.Until(ExpectedConditions.ElementIsVisible(firstName));
```

Example 2:

```csharp
        public AddAdressPage(IWebDriver browser)
        {
            driver = browser;
            var wait = new WebDriverWait(driver, TimeSpan.FromSeconds(2));
            wait.Until(ExpectedConditions.ElementIsVisible(firstName));
        }

        private By firstName = By.Id("address_first_name");
        private IWebElement TxtFirstName => driver.FindElement(firstName);
```

### **Session 6**
**Scope:** This session scope is to handle grids in our automation script
At this point, we have added an address, but the application has also implemented the delete functionality. This can be done in the Addresses page.
Within this page we have a grid that contains a list of addresses(table row > tr) and each address contains it's details(table data > td) and the posibility to view/edit/delete it.
After clicking delete button, an alert appears. Selenium Webdriver manipulate(accept/dismiss/get text) the alert and accept it in our case:

```csharp
		driver.SwitchTo().Alert().Accept();
```

Selenium Webdriver gives us the posibility to chain elements and create a structure father child:
```csharp
		driver.FindElement("father").FindElement(By.CssSelector("child")).FindElement(By.CssSelector("grandchild"));
```

We need to iterate the list to identify the address that we have added and to delete it. There are multiple ways to do it, but the preferred one is documented below: 
```csharp
	private By addresses = By.CssSelector("tbody tr");
        private IList<IWebElement> LstAddresses => driver.FindElements(addresses);
		
	private  By delete = By.CssSelector("[data-method='delete']");
        private IWebElement BtnDeleteV2 => LstAddresses.FirstOrDefault(element => element.Text.Contains("**hotel name**"))?
                                                        .FindElement(delete);
														
	public void DeleteAddress()
        {
            BtnDeleteV2.Click();
            driver.SwitchTo().Alert().Accept();
        }
```

The other ones would be:
```csharp
	private  By delete = By.CssSelector("[data-method='delete']");
		//**not a good idea because it would always take the first from the list
        private IWebElement BtnDelete => LstAddresses[0].FindElement(delete);


	public void DeleteAddressV1()
        {
            BtnDelete.Click();
            driver.SwitchTo().Alert().Accept();
        }
		
        public void DeleteAddressV2(AddAddressBO addAddressBo)
        {
            foreach (var address in LstAddresses)
            {
                if (address.Text.Contains(addAddressBo.TxtFirstName))
                {
                    address.FindElement(delete).Click();
                    driver.SwitchTo().Alert().Accept();
                    break;
                }
            }
        }    
```


And the methods could be called in test classes:
```csharp

	    [TestClass]
	    public class AddressesTest
	    {
	        private IWebDriver driver;
	        private AddressesPage addressesPage;
	
	        [TestInitialize]
	        public void SetUp()
	        {
	            driver = new ChromeDriver();
	            
	            driver.Manage().Window.Maximize();
	            driver.Navigate().GoToUrl("http://a.testaddressbook.com/");
	            var loginPage = new LoginPage(driver);
	            loginPage.menuItemControl.NavigateToLoginPage();
	            loginPage.LoginApplication("test@test.test", "test");
	
	            var homePage = new HomePage(driver);
	            //Implicit Wait
	            //driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(1);
	
	            addressesPage = homePage.NavigateToAddressesPage();
	            var addAddressPage = addressesPage.NavigateToAddAddressPage();
	            addAddressPage.AddAddress(new AddAddressBO());
	            var addressDetails =  addAddressPage.NavigateToAddressDetailsPage();
	            addressesPage = addressDetails.NavigateToAddressesPage();
	        }
	
	        [TestMethod]
	        public void Should_Delete_Address_V1()
	        {
	            addressesPage.DeleteAddress1();
	            string notice = "Address was successfully destroyed.";
	            Assert.AreEqual(notice, addressesPage.NoticeText);
	        }
	
	        [TestMethod]
	        public void Should_Delete_Address_V2()
	        {
	            addressesPage.DeleteAddressV2(new AddAddressBO());
	            string notice = "Address was successfully destroyed.";
	            Assert.AreEqual(notice, addressesPage.NoticeText);
	        }
	
	        [TestMethod]
	        public void Should_Delete_Address_V3()
	        {
	            addressesPage.DeleteAddressV3();
	            string notice = "Address was successfully destroyed.";
	            Assert.AreEqual(notice, addressesPage.NoticeText);
	        }
	
	        [TestCleanup]
	        public void CleanUp()
	        {
	            driver.Quit();
	        }
	}		
```



### **Session 7** 
**Scope:** This session scope is to handle the menu
The menu is present in all the app pages and we need to create a single repository where the menu elements can be stored.
This is a shared component and we need to call it in all of our page objects.
The first step is to create a class named MenuItemControl. This class will containt all menu elements.

```csharp

	    public class MenuItemControl
	    {
	        public IWebDriver driver;
			
			public MenuItemControl(IWebDriver browser)
	        {
	            driver = browser;
	        }
			
		private By home = By.CssSelector("");
	        private IWebElement BtnHome => driver.FindElement(home);
	    
	        private By signIn = By.Id("sign-in");
	        private IWebElement BtnSignIn => driver.FindElement(signIn);
	
	
	        private By addresses = By.CssSelector("");
	        private IWebElement BtnAddresses => driver.FindElement(addresses);
	
	        private By signOut = By.CssSelector("");
	        private IWebElement BtnSignOut => driver.FindElement(signOut);
	
	        private By useremail = By.CssSelector("span[data-test='current-user']");
	        private IWebElement LblUserEmail=>driver.FindElement(useremail);
		}
		
```

The application has 2 contexts, but this menu cannot be used from both perspectives: logged out and logged in.
Let's identify the elements used in this contextes:
			- logged out: Home, Sign in
			- logged in: Home, Addresses, Sign out and User email
The common webelement for both contextes is Home.
Now that we have identified what we have, we need to create 2 classes for out contextes: LoggedOutMenuItemControl and LoggedInMenuItemControl that will inhirit the MenuItemControlClass.
And we need to move the elements to the according classes. We also need to move the navigation logic to this classes.

```csharp

	    public class MenuItemControl
	    {
	        public IWebDriver driver;
	
	        public MenuItemControl(IWebDriver browser)
	        {
	            driver = browser;
	        }
	
	        private By home = By.CssSelector("");
	        private IWebElement BtnHome => driver.FindElement(home);
	    }
		
```

```csharp

	    public class LoggedOutMenuItemControl: MenuItemControl
	    {
	
	        private By signIn = By.Id("sign-in");
	        private IWebElement BtnSignIn => driver.FindElement(signIn);
	
	        public LoggedOutMenuItemControl(IWebDriver browser) : base(browser)
	        {
	        }
	
	        public LoginPage NavigateToLoginPage()
	        {
	            BtnSignIn.Click();
	            return new LoginPage(driver);
	        }
	    }
		
```

```csharp

	    public class LoggedInMenuItemControl: MenuItemControl
	    {
	        private By addresses = By.CssSelector("");
	        private IWebElement BtnAddresses => driver.FindElement(addresses);
	
	        private By signOut = By.CssSelector("");
	        private IWebElement BtnSignOut => driver.FindElement(signOut);
	
	        private By useremail = By.CssSelector("span[data-test='current-user']");
	        private IWebElement LblUserEmail=>driver.FindElement(useremail);
	
	        public LoggedInMenuItemControl(IWebDriver browser) : base(browser)
	        {
	        }
	
	        public string UserEmailText => LblUserEmail.Text;
	    }
		
```

Let's used in the LoginPage.cs and LoginTests.cs

```csharp

	     public class LoginPage
	     {
	        private IWebDriver driver;
			
			//**reference the menu item control**
	        public LoggedOutMenuItemControl menuItemControl => new LoggedOutMenuItemControl(driver);
	
	        public LoginPage(IWebDriver browser)
	        {
	            driver = browser;
	        }        
	
	        private By email = By.Id("session_email");
	        private IWebElement TxtUsername()
	        {
	            return driver.FindElement(email);
	        }
	
	        private IWebElement TxtPassword()
	        {
	            return driver.FindElement(By.Id("session_password"));
	        }
	
	        private IWebElement BtnLogin()
	        {
	            return driver.FindElement(By.Name("commit"));
	        }
	        
	        public void LoginApplication(string username, string password)
	        {
	            var wait = new WebDriverWait(driver, TimeSpan.FromSeconds(2));
	            wait.Until(ExpectedConditions.ElementExists(email));
	            TxtUsername().SendKeys(username);
	            TxtPassword().SendKeys(password);
	            BtnLogin().Click();
	        }
	    }
		
```

```csharp

	    [TestClass]
	    public class LoginTests
	    {
	        private IWebDriver driver;
	        private LoginPage loginPage;
	
	
	        [TestInitialize]
	        public void SetUp()
	        {
	            driver = new ChromeDriver();
	            loginPage = new LoginPage(driver);
	            driver.Manage().Window.Maximize();
	            driver.Navigate().GoToUrl("http://a.testaddressbook.com/");
				//**use in the test**
	            loginPage.menuItemControl.NavigateToLoginPage();
	        }
	
	        [TestMethod]
	        public void Login_CorrectEmail_CorrectPassword()
	        {
	            loginPage.LoginApplication("test@test.test", "test");
	
	            var expectedResult = "test@test.test";
	            var homePage = new HomePage(driver);
	
	            Assert.AreEqual(expectedResult, homePage.menuItemControl.UserEmailText);
	        }
	
	        [TestMethod]
	        public void Login_IncorrectEmail_IncorrectPassword()
	        {
	            loginPage.LoginApplication("weor@hdsh.asdhg", "asd");
	
	            var expectedResult = "Bad email or password.";
	            var actualResults = driver.FindElement(By.XPath("//div[starts-with(@class, 'alert')]")).Text;
	
	            Assert.AreEqual(expectedResult, actualResults);
	        }
	
	        [TestCleanup]
	        public void CleanUp()
	        {
	            driver.Quit();
	        }
	    }
		
```

As said before, this can be put in every class, depending on the context.
