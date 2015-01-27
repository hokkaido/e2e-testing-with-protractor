#AngularJS - End-to-end testing with protractor


As developers concerned with the quality of our work, we have a multitude of tools and patterns at our disposal to ensure
that everything works as expected. In short, we test our software.

A typical web application has to integrate a variety of external services, database systems and APIs. There comes a point 
during the development of most large web applications where you want to test the functionality of the system as a whole. 

This is an area that is notoriously difficult to test with traditional methods such as unit tests and simple
mocks. A database can fail, an external service can return an invalid result and a new browser version might have introduced
a simple bug that we didn't know about when we initially wrote our code.

On top of that, one of the most important areas of software testing deals with the user-facing part of an application. 
After all, we build software not for ourselves, but for our clients and their customers. 

Luckily enough, there are existing solutions that help us deal with these problems in an automated and consistent way.

Now, this is a blog post about AngularJS. We use AngularJS and we test our code. In the following paragraphs I'd like to describe one of the ways how we test AngularJS applications at Liip.

##Enter protractor
Google has released a testing framework for AngularJS applications called Protractor that integrates existing technologies such as Selenium, Node.js and Jasmine and makes writing tests a breeze.

With protractor we can write tests that run inside an actual browser, against an existing website. We can test whether our website works as intended and we can catch and guard against unexpected errors.

If you know Selenium and Jasmine, getting started with Protractor should be pretty straight forward.

##Writing a simple test
Let's assume we want to test whether a login page displays an error message if we do not fill in the password field.
In protractor, this might look like this:

```javascript
describe('login page', function() {
  it('should display an error if the password field is empty', function() {
    browser.get('http://mysuperawesomepage.com/login');
    element(by.model('userName')).sendKeys('gandalf');
    element(by.id('btn-submit')).click();
    expect(element(by.css('.password-error')).isDisplayed()).toBe(true);
  });
});
```

So, what have we done here?

- We `describe` the page we want to test, in this case, our login page
- We describe the behavior we want to test, in this case `it` should display an error message if the password field is empty
- We visit (`get`) the page, in this case mysuperawesomepage.com/login
- We instruct protractor to fill in a user name and press the submit button.
- We test whether our expected error message is displayed and use Jasmine's expect syntax to achieve this.

 
##ElementFinder and Locators
As you can imagine, a large part of writing a test against a web site deals with finding and locating elements on a page and executing
actions such as clicking on them. Protractor helps us here with two constructs: ElementFinders and Locators.

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
If we only relied on `element` calls to structure our tests, our life gets progressively worse as the application grows. One change to an element's class name could force us to rewrite many of our tests.


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
And we can now use the LoginPage in a test like this:
 
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
 
We can now extend this principle to shared page components, such as headers, footers and directives. Once you've written

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
