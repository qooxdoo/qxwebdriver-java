# qxwebdriver-java

WebDriver testing support for qooxdoo desktop applications.

This is an open source project, led by one of the world's largest web hosts [1&1](http://www.1and1.com), with a vibrant community.

The goal of this project is to provide an API that facilitates writing [WebDriver](http://seleniumhq.org/docs/03_webdriver.html)-based interaction tests for [qx.Desktop applications](http://manual.qooxdoo.org/current/pages/desktop.html) by abstracting away the implementation details of qooxdoo widgets. Here's a quick example:

    QxWebDriver driver = new QxWebDriver(new FirefoxDriver());
    // Open the page and wait until the qooxdoo application is loaded
    driver.get("http://demo.qooxdoo.org/current/apiviewer/index.html");

    // Find the 'content' button in the tool bar
    By button = By.qxh("apiviewer.Viewer/*/[@label=Content]");
    Widget contentButton = driver.findWidget(button);
    // Click the button if it's not already selected
    if (!contentButton.isSelected()) {
      contentButton.click();
    }

    // Select the "data" item from the package tree
    By tree = By.qxh("apiviewer.Viewer/*/apiviewer.ui.PackageTree");
    Selectable packageTree = (Selectable) driver.findWidget(tree);
    packageTree.selectItem("data");

## See it in action

Clone the repo, then run

    mvn verify

This will run the included example tests against the [Widget Browser](http://demo.qooxdoo.org/current/widgetbrowser/) using Firefox, so you must have Firefox installed and a working internet connection.

## Widget Interfaces

QxWebDriver provides a set of Widget classes similar to WebDriver's support classes, each of which implements _org.oneandone.qxwebdriver.ui.Widget_ or one or more of the interfaces inheriting from it, such as _Selectable_ or _Scrollable_. These interfaces allow complex actions to be performed by relatively few API calls.

Widgets are obtained by calling _QxWebDriver.findWidget(by)_. where _by_ is any locator strategy that finds a DOM element which is part of a qooxdoo widget. _findWidget_ will determine the qooxdoo class of the widget, its inheritance hierarchy and the interfaces it implements, and use this information to decide which _Widget_ implementation to return.

Similar to _WebElement.findElement_, _Widget.findWidget_ will restrict the search to children of the current widget.

### Examples

The best way to learn about the various Widget interfaces is to check out the sample integration tests in [src/test/java](https://github.com/qooxdoo/qxwebdriver-java/tree/master/src/test/java).

## Locating Widget Elements

WebDriver's built-in _"By"_ strategies like _By.xpath_ or _By.className_ generally don't work too well with the entangled and dynamic DOM structures generated by qx.Desktop applications. The _By.qxh_ strategy (qx for qooxdoo, h for hierarchy) provides an alternative approach that searches for elements by using JavaScript to traverse the application's widget hierarchy.

For example, the [qooxdoo Feed Reader](http://demo.qooxdoo.org/current/feedreader/)'s UI hierarchy looks like this (easily determined by opening the Feed Reader in the [Inspector](http://demo.qooxdoo.org/current/inspector/)):

    qx.ui.root.Application
    - qx.ui.container.Composite
      - feedreader.view.desktop.Header
      - feedreader.view.desktop.ToolBar
        - qx.ui.toolbar.Button
      [...]

A qxh locator that finds a toolbar button with the label _Reload_ could look like this:

    By by = By.qxh("child[0]/feedreader.view.desktop.ToolBar/[@label=Reload]");

As you can see, the syntax is similar to XPath, consisting of location steps separated by slashes. While searching, each location step selects a widget which will be used as the root for the rest of the search.

### Supported Steps

*   **foo.bar.Baz** A string containing dots will be interpreted as the class name of a widget. Uses _instanceof_ internally so inheriting widgets will be found as well.
*   **child[N]** Signifies the Nth child of the object returned by the previous step.
*   **[@attrib{=value}]** Selects a child that has a property _attrib_ which has a value of _value_. "Property" here covers both qooxdoo as well as generic JavaScript properties. As for the values, only string comparisons are possible, but you can specify a RegExp which the property value will be matched against. _toString()_ is used to compare non-String values.
*  __\*__ is a wildcard operator. The wildcard can span zero to multiple levels of the object hierarchy. This saves you from always specifying absolute locators, e.g. the example above could be rewritten as


    By by = By.qxh("*/[@label=Reload]");

This will recursively search from the first level below the search root for an object with a _label_ property that matches the regular expression _/Reload/_. As you might expect, these recursive searches take more time than other searches, so it is good practice to be as specific in your locator as possible.

Note that the qxh strategy will only return the **first match** for the locator expression, so it can't be used with _WebDriver.findElements_.

### Relative Locators

The root node where a _By.qxh_ locator will begin searching is determined by its context: When used with _QxWebDriver.findWidget_, the children of the qooxdoo application's root widget will be matched against the first step.
When used with _Widget.findWidget_, the widget itself will be the root node for the search.

## Getting Started
The [Getting Started](https://github.com/qooxdoo/qxwebdriver-java/wiki/Getting-Started) tutorial explains how to set up and run qxwebdriver tests using Maven and Firefox.

## Extending qxwebdriver

_findWidget_ uses a _WidgetFactory_ to determine which Widget class to instantiate. An alternative class implementing _WidgetFactory_ and probably extending _DefaultWidgetFactory_ can be passed to the _QxWebDriver_ constructor to support custom widgets.

## Browser Support

In theory, QxWebDriver should work with any WebDriver that implements _JavascriptExecutor_. In practice, FirefoxDriver, ChromeDriver and IEDriver (with IE9) all work well. The current OperaDriver frequently throws exceptions when trying to execute JavaScript, rendering it basically useless for the purpose of testing qx applications.

## Project Status

qxwebdriver is still experimental. The API is subject to change without notice, not all of qooxdoo's built-in widgets are supported and there is much still to be optimized. Don't let that stop you from playing with it, giving us feedback on [the usual channels](http://qooxdoo.org/community/) and sending pull requests. Thanks!

### Still To Come

* Support for _qx.ui.table.Table_
* Maven/Jenkins integration
* Access to qooxdoo's logging and global error handling facilities
* ...
