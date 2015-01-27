#AngularJS - End-to-end testing with Protractor

We use AngularJS, and as developers concerned with the quality of our work, we employ a multitude of tools and patterns to ensure that everything works as expected. In short, we test our software.

The goal of this post is to give you a short overview about some of our experiences with testing AngularJS applications, and I'll focus exclusively on end-to-end testing.

##End-to-end testing

Modern web applications have to integrate a variety of external services, database systems and APIs. On top of that, they have to accomodate an ever-shifting landscape of devices and browsers. 

This is an area that is notoriously difficult or even outright impossible to test with traditional methods such as unit tests and simple mocks. A database can fail, an external service can return an invalid result and a new browser version might have introduced a simple bug that we didn't know about when we initially wrote our code.

This is where end-to-end testing comes into play. We want to test our application as a whole and make sure that it works as expected. We want to test our entire application, starting from the user interface down to individual subsystems, backend and external services.

It's important to note that end-to-end testing is no panacea, we employ it alongside other testing methods, but the high level and broad scope of end-to-end testing can help tremendously when developing complex web applications.

##Enter Protractor
Google has released an end-to-end testing framework for AngularJS applications called  [Protractor](http://angular.github.io/protractor/) that integrates existing technologies such as Selenium, Node.js and Jasmine and makes writing tests a breeze.

With Protractor we can write tests that run inside an actual browser, against an existing website.

If you already know Selenium and Jasmine, getting started with Protractor should be pretty straight forward.

##Installation

##Writing a simple test
Protractor expects your tests to be written in so-called spec files. Spec is simply another word for test. These spec files are written using the syntax of your test framework, and the Protractor API. Out of the hood, Protractor uses [Jasmine](http://jasmine.github.io/1.3/introduction.html), but it also has tentative support for [Mocha](http://mochajs.org) and [Cucumber](http://cukes.info).

Let's assume we want to test whether a login page displays an error message if we do not fill in the password field.
In protractor, we'd create a spec file (`login_spec.js`) for it that might look like this:

```javascript
describe('login page', function() {
  it('should display an error if the password field is empty', function() {
  
    // Visit the login page
    browser.get('http://mysuperawesomepage.com/login');
    
    // Find the element that matches ng-model="userName" and type 'gandalf' into it.
    element(by.model('userName')).sendKeys('gandalf');
    
    // Find the submit button and click it
    element(by.id('btn-submit')).click();
    
    // Check whether our error message is displayed
    expect(element(by.css('.password-error')).isDisplayed()).toBe(true);
  });
});
```

So, what have we done here?

The `describe` call is from Jasmine, and we use it to describe the page we want to test, in this case, our login page. The `it` call is also from Jasmine, and we use it to describe the scope of our test. In this case the login page should display an error message if the password field is empty.

`browser` is a global variable exposed by Protractor, and we use it to visit our (`get`) login page.

`element` and `by` are also globals created for us by Protractor. We can use them to find and interact with elements on the page.

`expect` is again from Jasmine, we test whether our expected error message is displayed.
 
##ElementFinder and Locators
As you can imagine, a large part of writing a test against a web site deals with finding and locating elements on a page and executing actions such as clicking on them. Protractor helps us here with two constructs: ElementFinders and Locators.

Some examples:

If you want to find the first (or only) element by using a css selector:

`element(by.css('input.username'));`

If you want to find all elements on a by using a css selector

`element.all(by.css('a.btn'));`

If you want to find an element on the page by using its id

`element(by.id('company-name'));`

If you want to find all elements that have an ng-bind="currency" attribute

`element.all(by.binding('currency'));`

If you want to find an element that uses ng-model="selectedAlbum":

`element(by.model('selectedAlbum'));`

Once you have an ElementFinder, you can trigger actions:

If you want to get an element's text value:

`element.all(by.css('a.home-page')).getText();`

If you want to get an element's attribute:

`element(by.css('a.home-page')).getAttribute('target');`

###Chaining Elements

You can easily chain element calls.

`element.all(by.css('li')).element(by.;`

###ElementFinder = Promises

##Element Finders are
 
##Organizing your code: Page objects
If we only relied on `element` calls to structure our tests, our life would get progressively worse as the application grows. We'd have to duplicate a lot of code for components that are used across different pages. One change to an element's class name could force us to rewrite many of our tests.

We use a design pattern called *page object* to overcome this problem. It is described in more detail by [Martin Fowler](http://martinfowler.com/bliki/PageObject.html). The gist of it is very simple.


```javascript
var LoginPage = function() {
  this.username = element(by.model('username'));
  this.password = element(by.model('password'));
  this.loginButton = element(by.id('btn-login'));
  this.passwordRequiredError = element(by.css('error-password-required'));
    
  this.visit = function() {
    browser.get('http://mysuperawesomepage.com/login');
  }
    
  this.setUsername = function(username) {
    this.username.clear();
    this.username.sendKeys(username);
  }
    
  this.setPassword = function(password)
    this.password.clear();
    this.password.sendKeys(password);
  }
    
  this.login = function() {
    this.loginButton.click();
  }
};
module.exports = LoginPage;

```
We can now use the LoginPage page object in a test like this:
 
```javascript
var LoginPage = require('login-page');

describe('login page', function() {
  it('should display an error message if the password field is empty', function() {
    var page = new LoginPage();
    page.visit();
    page.userName = 'gandalf';
    page.login();
    expect(page.passwordRequiredError.isDisplayed).toBe(true);
  });
});  
```
 
We extend this principle to shared page components, such as headers, footers and AngularJS directives. 

```javascript
var HomePage = function() {
  this.header = new Header();
  this.sideBar = new SideBar();
  this.footer = new Footer();
  ...
};
module.exports = HomePage;
```

We often unify common methods in a base `Page`class that other page objects can inherit from:

```javascript
var Page = function() {
  this.clearAndType = function(element, text) {
    element.clear();
    element.sendKeys(text);
  };
  
  //...
};  
module.exports = Page;
```
