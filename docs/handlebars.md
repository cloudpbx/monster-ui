title: Using Handlebars for your templates

# Handlebars in Monster-UI
In order to add dynamic data in our views, we needed a templating engine. And Handlebars is the templating engine we chose. In this document, we'll explain how to use it, and how to use some helpers that can be very helpful for certain tasks.

* [How to use it](#how-to-use-it)
* [Going Further](#going-further)
* [Listing of different helpers](#listing-of-different-helpers)
  - [coalesce](#coalesce)
  - [compare](#compare)
  - [debug](#debug)
  - [ifInArray](#ifinarray)
  - [isPrivLevelAdmin](#isprivleveladmin)
  - [isSuperDuper](#issuperduper)
  - [formatMacAddress](#formatmacaddress)
  - [formatPhoneNumber](#formatphonenumber)
  - [formatPrice](#formatprice)
  - [getUserFullName](#getuserfullname)
  - [monsterCheckbox](#monstercheckbox)
  - [monsterNumberWrapper](#monsternumberwrapper)
  - [monsterPanelText](#monsterpaneltext)
  - [monsterRadio](#monsterradio)
  - [monsterSignalIndicator](#monstersignalindicator)
  - [monsterSwitch](#monsterswitch)
  - [monsterText](#monstertext-obsolete-use-monsterpaneltext-where-possible)
  - [replaceVar](#replacevar)
  - [toFriendlyDate](#tofriendlydate)
  - [toLowerCase](#tolowercase)
  - [tryI18n](#tryi18n)

## How to use it
Let's start with an easy example. We want to display a banner displaying `Hello Georges!` whenever the User georges logs in. The HTML to do that could be something like this:

hello.html
```html
<div class="banner-hello">
  <span>Hello Georges!</span>
</div>
```
This hello.html file is a template. In order to add it somewhere in your UI, you would do `container.append(monster.template(self, 'hello'))` and it would add this HTML inside the container.

Now let's add some dynamic data, since that's why we need to use Handlebars in the first place! The problem with our template above, is that we need to say Hello to the user, and we don't want to display Hello Georges! but Hello followed by the first name of the user. In order to do so, we need to have a variable inside this template. We know that in order to give a variable to our template we need to do:

app.js
```javascript
var dataTemplate = {
    firstName: 'Pedro'
  },
  template = monster.template(self, 'hello', dataTemplate);
```

hello.html
```html
<div class="banner-hello">
  <span>Hello {{firstName}}!</span>
</div>
```
It's almost exactly the same template as above, except that we're now using the `firstName` variable, and we can now display it properly for all the users.

## Going further
We require all the templates to be internationalizable. We don't force you to create more than the en-US.json file, but it's important to have the capability to add more languages later on. If you want a better understanding on how to use the i18n, please go visit [this link][i18n]

## Listing of different helpers
Handlebars comes with a lot of helpers already built, you can see the list [here](http://handlebarsjs.com/). It includes basic helpers such as each, if, unless, etc...

Now we'll check which helpers we created to help us specifically with some common problems we encountered.

#### coalesce
This helper will return the first non nullish (`null` or `undefined`) element to be found from a list of 2 or more values. If all of them are nullish, then it will return `null`.

```handlebars
{{coalesce val1 val2 ... valN}}
```

#### compare
There is no way natively in Handlebars to compare if a string is equals to "test2" for example. Although it makes sense to try and avoid doing logic in the views, there are some places where we need this logic in the view instead of adding tons of code in the JS. The compare helper is here to help us with it.

In order to use it, you need to give 3 arguments:
* the left value,
* the operator, needs to be in this list: ['==', '===', '!=', '!==', '<', '>', '<=', '>=', 'typeof'],
* the right value

Let's see an example:

example.html
```handlebars
{{compare role "===" "admin"}}
  <div class="admin-welcome">You're an admin! You must be so cool!</div>
{{else}}
  <div class="peon-welcome">You're a user! That's cool.</div>
{{/compare}}
```
In the above example, we check if the role of a user is `"admin"` and if it is we display a special div, otherwise we display another one. We can use it the same way to compare values, for example `{{compare moneyLeft ">=" 10000}}You're so rich...{{/compare}}`. You get the idea!

#### debug
The debug helper is really helpful to debug your templates. Sometimes you lose track of the current context in the template, and dropping a `{{debug}}` will output the current context in the console.
You can also add an argument after it to output a specific object. Here's an example:

example.html
```handlebars
{{debug}} <!-- Would output the entire context, containing users and anything else -->
{{debug users}} <!-- Would output the entire users object}} -->
{{#each users}}
  {{debug}} <!-- Would output the object representing the current user -->
  <div class="user-div">
    <span class="user-first-name">{{firstName}}</span>
    <span class="user-last-name">{{lastName}}</span>
  </div>
{{/each}}
```

#### ifInArray
This helper will check if an array contains a specific value and display the related HTML.

example.html
```handlebars
<input type="checkbox"{{#ifInArray 'VP8' media.video.codecs}} checked="checked"{{/ifInArray}} value="{{codec}}"/>
```
Here we tick the checkbox depending on the 'VP8' video codec being in the list of available codecs.

#### isPrivLevelAdmin
This is a predetermined conditional helper letting you render content only if the privilege level of the user logged in or passed as argument is `admin`.

```handlebars
{{#isPrivLevelAdmin optionalUserObject}}
...
{{/isPrivLevelAdmin}}
```

#### isSuperDuper
This is a predetermined conditional helper letting you render content only if the account logged in or passed as argument has the `superduper_admin` flag set to `true`.

```handlebars
{{#isSuperDuper optionalAccountObject}}
...
{{/isSuperDuper}}
```

#### formatMacAddress
The formatMacAddress helper formats a string into a string representation of a MAC address, using colons as separator.

```handlebars
{{formatMacAddress 'E3.FF.34.9A.71.BC'}}
```

#### formatPhoneNumber
The formatPhoneNumber helper can display a phone-number with proper formatting automatically. It's useful because a lot of numbers are stored in the database with a format such as `+14151993821` and we want to display them to the user like `+1 (415) 199-3821`.

example.html
```handlebars
<div class="user-div">
  <span class="user-first-name">{{firstName}}</span>
  <span class="user-last-name">{{lastName}}</span>

  <div class="user-phone-number">{{formatPhoneNumber phoneNumber}}</div>
</div>
```

#### formatPrice
The formatPrice helper can display a price with the proper amount of digits after the decimal point as well as the currency used by the UI. It's useful because prices can have a various number of digits, but should be displayed in a consistent way.
The example below will round off the price to two (2) digits, and display it with two (2) digits even if it has less than that (i.e. "6" -> "6.00", "6.666" -> "6.67").

```handlebars
<div class="item-row">
  <span class="item-name">{{name}}</span>
  <span class="item-price">{{formatPrice price 2}}</span>
</div>
```

If you do not want the currency to be displayed, you'll have to specify a third argument:
```handlebars
<div class="item-row">
  <span class="item-name">{{name}}</span>
  <span class="item-price">{{formatPrice price 2 false}}</span>
</div>
```

#### getUserFullName
This helper will display the full name of the user object passed as argument, following the [internationalization][i18n] rules that are currently active. If no argument is provided, it will return the full name of the user that is currently logged in. The user parameter should be a plain object that has the properties `first_name` and `last_name`.

```handlebars
{{getUserFullName optionalUserObject}}
```

#### monsterCheckbox
This helper allows you to generate a pretty checkbox from a simple checkbox input.

```handlebars
{{#monsterCheckbox}}
  <input type="checkbox" />
{{/monsterCheckbox}}
```

Any property (class, id, checked, data-something ,etc...) set on the input will be conserved.
You can also provide parameters to define a label, set its positioning, and set the size of the checkbox. All optional.

```handlebars
{{#monsterCheckbox "large-checkbox" "prepend-label" "My Checkbox's label"}}
  <input type="checkbox" />
{{/monsterCheckbox}}
```

#### monsterNumberWrapper
This helper will generate a wrapper around a phone number that will automatically format that number based on user preferences, as well as showing a flag if the phone number was a valid phone number from a known country.

example.html
```handlebars
{{#monsterNumberWrapper this.phoneNumber}}{{/monsterNumberWrapper}}
```

#### monsterPanelText
This helper is a re-design of the monsterText wrapper. It allows developer to display "informational" messages nicely.

There are a few parameters available to customize this helper.

First one is the `title`, this new panel includes a section for a title. You should use a short text like "Warning!" or "Information".

The second parameter is the `style`. There are 4 different styles for this panel, `info` (blue), `success` (green), `warning` (yellow), `dange`" (red).

Finally you can add a `className` as a third parameter. This is helpful if you need to customize the helper yourself. You can also use `fill-width` as a class name to automatically fill the parent's div width with the helper. Without that it will be set to a fixed size.

Here are some examples on how to use this helper.

```handlebars
{{#monsterPanelText 'Warning!' 'warning' 'fill-width'}}
  You should see a warning message here!
{{/monsterPanelText}}
{{#monsterPanelText 'Help' 'info'}}
  <p>This is an informational message</p>
{{/monsterPanelText}}
{{#monsterPanelText 'Error!' 'danger' 'fill-width'}}
  You should see an error message here
{{/monsterPanelText}}
{{#monsterPanelText 'Congratulations!' 'success' 'fill-width'}}
  You should see a success message here
{{/monsterPanelText}}
```

#### monsterRadio
Similar to monsterCheckbox, this helper allows you to generate a pretty radio button from a simple radio input.

```handlebars
{{#monsterRadio}}
  <input type="radio" name="myRadioName" />
{{/monsterRadio}}
```

Any property (class, id, checked, data-something ,etc...) set on the input will be conserved.
You can also provide parameters to define a label, set its positioning, and set the size of the radio. All optional.

```handlebars
{{#monsterRadio "large-radio" "prepend-label" "My Radio Button's label"}}
  <input type="radio" name="myRadioName" />
{{/monsterRadio}}
```

#### monsterSignalIndicator
This helper will generate a signal strength indicator using a pair of parameters to indicate the strength and type of signal to display.

```handlebars
{{#monsterSignalIndicator strength}}
  {{label}}
{{/monsterSignalIndicator}}
```

* strength: Number from 0 to 4
* label: String

#### monsterSwitch
This helper allows you to generate a switch from a simple checkbox.

```handlebars
{{#monsterSwitch}}
  <input type="checkbox" />
{{/monsterSwitch}}
```

Any property (class, id, checked, data-something ,etc...) set on the input will be conserved.

#### monsterText (obsolete, use monsterPanelText where possible)
This helper allows you to generate a wrapper around your text, with a pre-set design used in many different places in the Monster-UI, so it looks consistent with the rest of the UI.

First argument is optional and defaults to 'info'. Choices are 'info', 'question', 'error', 'warning'. Error will display a wrapper of the red color, Warning will display a wrapper of the yellow color, and question will show a question mark instead of the standard info icon.

Second argument is optional and let you define a className that will be added to the main wrapper so you can apply some CSS rules to it.

```handlebars
{{#monsterText}} Monster is <h3>awesome</h3> {{/monsterText}}
{{#monsterText 'error'}} Monster is <strong>NOT</strong> awesome {{/monsterText}}
{{#monsterText 'warning' 'myClassName'}} Monster could be awesome {{/monsterText}}
{{#monsterText 'question'}} Is Monster Awesome?{{/monsterText}}
```

#### replaceVar
This helper allows you to replace a variable inside another variable with Handlebars. It's useful for i18n keys.
You can see a very good example about this helper [here][i18n_templates].

#### toFriendlyDate
This helper allows you to display time in a customizable format. It take a mandatory Gregorian timestamp in parameter and returns a formatted date:

```handlebars
{{toFriendlyDate gregorianTimestamp}}
```
Output: 12/01/14 12:43PM

If you want to display a time in another format, you can specify an optional parameter:

```handlebars
{{toFriendlyDate gregorianTimestamp 'DD-MM-year hh:mm:ss12h'}}
```
Output: 12-01-2014 12:43:23PM

In bonus, you can set the optional parameter to `short` to simply display the date:

```handlebars
{{tofriendlyDate gregorianTimestamp 'short'}}
```
Output: 12/01/2014

This helper will search for the following strings and replace them by the corresponding values:
* year: full year
* YY: last 2 digits of the year
* month: month of the year in letters
* MM: month as a 2 digits number
* day: day of the week in letters
* DD: date of the day
* hh: hours
* mm: minutes
* ss: seconds
* 12h: use the 12h format (if not specified, the 24h format is used)

#### toLowerCase
This helper is pretty simple, and is basically executing the toLowerCase() JS function, to a string in a template. For example it would transform `Giorgio` in `giorgio`.

example.html
```handlebars
<div class="user-div">
  <span class="small-user-first-name">{{toLowerCase firstName}}</span>
</div>
```

#### tryI18n
If you want to display data from the back-end, often times you'll try to parse the text and have a better text displayed instead.
But what if the back-end moves faster and add more keys? You still want to be able to display text for that key. This is what tryI18n is for.
Basically, it will check if a key has a translation in a JSON object. If it has one, it will display the friendly text, if not, it will display the variable name!


Let's say we have a i18n file like:
```JSON
{
  "demoHandlebars": {
    "test_1": "Test 1",
    "test_2": "Test 2",
    "test_3": "Test 3"
  }
}
```

and a HTML template named 'templateName' like:
```handlebars
{{#each keys}}
  {{ tryI18n i18n.demoHandlebars this }}<br/>
{{/each}}
```

And we use this in JS:
```javascript
var obj = { keys: ['test_1','test_2','test_3'] };

monster.template(self, 'templateName', obj);
```

The final output would be something like

```text
Test 1

Test 2

Test 3
```

[i18n]: internationalization.md
[i18n_templates]: internationalization.md#in-html-templates
