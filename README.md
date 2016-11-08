# International Telephone Input [![Build Status](https://travis-ci.org/jackocnr/intl-tel-input.svg)](https://travis-ci.org/jackocnr/intl-tel-input)
A jQuery plugin for entering and validating international telephone numbers. It adds a flag dropdown to any input, detects the user's country, displays a relevant placeholder and provides formatting/validation methods.

<img src="https://raw.github.com/jackocnr/intl-tel-input/master/screenshot.png" width="424px" height="246px">

If you like it, please consider making a donation, which you can do from [the demo page](http://intl-tel-input.com).

## Table of Contents

- [Demo and Examples](#demo-and-examples)
- [Features](#features)
- [Browser Compatibility](#browser-compatibility)
- [Getting Started](#getting-started)
- [Recommended Usage](#recommended-usage)
- [Options](#options)
- [Public Methods](#public-methods)
- [Static Methods](#static-methods)
- [Events](#events)
- [Utilities Script](#utilities-script)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Attributions](#attributions)


## Demo and Examples
You can view a live demo and some examples of how to use the various options here: http://intl-tel-input.com, or try it for yourself using the included demo.html.


## Features
* Automatically select the user's current country using an IP lookup
* Automatically set the input placeholder to an example number for the selected country
* Navigate the country dropdown by typing a country's name, or using up/down keys
* Handle phone number extensions
* The user types their national number and the plugin gives you the full standardized international number
* Full validation, including specific error types
* Retina flag icons
* Lots of initialisation options for customisation, as well as public methods for interaction


## Browser Compatibility
| Chrome | FF  | Safari | IE  | Chrome Android | Mobile Safari | IE Mob |
| :----: | :-: | :----: | :-: | :------------: | :-----------: | :----: |
|    ✓   |  ✓  |    ✓   |  8  |      ✓         |       ✓       |     ✓  |


## Getting Started
1. Download the [latest release](https://github.com/jackocnr/intl-tel-input/releases/latest), or better yet install it with [npm](https://www.npmjs.com/) or [Bower](http://bower.io)

2. Include the stylesheet
  ```html
  <link rel="stylesheet" href="path/to/intlTelInput.css">
  ```

3. Override the path to flags.png in your CSS
  ```css
  .iti-flag {background-image: url("path/to/flags.png");}
  ```
  _Update: you will now also need to override the path to flags@2x.png (for retina devices). The best way to do this is to copy the media query at the end of [intlTelInput.scss](https://github.com/jackocnr/intl-tel-input/blob/master/src/css/intlTelInput.scss) and update the path._

4. Add the plugin script and initialise it on your input element
  ```html
  <input type="tel" id="phone">

  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
  <script src="path/to/intlTelInput.js"></script>
  <script>
    $("#phone").intlTelInput();
  </script>
  ```

5. **Recommended:** initialise the plugin with the `utilsScript` option to enable formatting/validation, and to allow you to extract full international numbers using `getNumber`.


## Recommended Usage
We highly recommend you load the included utils.js using the `utilsScript` option. Then even when `nationalMode` or `separateDialCode` is enabled, the plugin is built to always deal with numbers in the full international format (e.g. "+17024181234") and convert them accordingly. I recommend you get, store, and set numbers exclusively in this format for simplicity.

You can always get the full international number (including country code) using `getNumber`, then you only have to store that one string in your database (you don't have to store the country separately), and then the next time you initialise the plugin with that number it will automatically set the country and format it according to the options you specify (e.g. if you enable `nationalMode` it will automatically remove the international dial code for you).


## Options
Note: any options that take country codes should be [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) codes  

**allowDropdown**  
Type: `Boolean` Default: `true`  
Whether or not to allow the dropdown. If disabled, there is no dropdown arrow, and the selected flag is not clickable. Also we display the selected flag on the right instead because it is just a marker of state.

**~~autoFormat~~ [REMOVED]**  
Automatically format the number as the user types. Unfortunately this had to be removed for the reasons listed here: [#346 Disable and remove autoFormat feature](https://github.com/jackocnr/intl-tel-input/issues/346).

**autoHideDialCode**  
Type: `Boolean` Default: `true`  
If there is just a dial code in the input: remove it on blur or submit, and re-add it on focus. This is to prevent just a dial code getting submitted with the form. Requires `nationalMode` to be set to `false`.

**autoPlaceholder**  
Type: `String` Default: `"polite"`  
Set the input's placeholder to an example number for the selected country (you can specify the number type using the `numberType` option). By default it is set to `"polite"`, which means it will only set it if the input doesn't already have a placeholder attribute. You can also set it to `"aggressive"`, which will replace any existing placeholder, or `"off"`. Requires the `utilsScript` option.

**customPlaceholder**  
Type: `Function` Default: `null`  
Change the placeholder generated by autoPlaceholder. Must return a string.

```js
customPlaceholder: function(selectedCountryPlaceholder, selectedCountryData) {
  return "e.g. " + selectedCountryPlaceholder;
}
```

**dropdownContainer**  
Type: `String` Default: `""`  
Expects a jQuery selector e.g. `"body"`. Instead of putting the country dropdown next to the input, append it to the element specified, and it will then be positioned absolutely next to the input using JavaScript. This is useful when the input is inside a container with `overflow: hidden`. Note that the absolute positioning can be broken by scrolling, so it will automatically close on the `window` scroll event. If you have a different scrolling element that is causing problems, simply listen for the scroll event on that element, and trigger `$(window).scroll()` e.g.

```js
$("#scrollingElement").scroll(function() {
  $(window).scroll();
});
```

**excludeCountries**  
Type: `Array` Default: `undefined`  
Don't display the countries you specify.

**formatOnInit**  
Type: `Boolean` Default: `true`  
Format the input value during initialisation.

**geoIpLookup**  
Type: `Function` Default: `null`  
When setting `initialCountry` to `"auto"`, you must use this option to specify a custom function that looks up the user's location. Also note that when instantiating the plugin, we now return a [deferred object](https://api.jquery.com/category/deferred-object/), so you can use `.done(callback)` to know when initialisation requests like this have completed.

Here is an example using the [ipinfo.io](http://ipinfo.io/) service:  
```js
geoIpLookup: function(callback) {
  $.get("http://ipinfo.io", function() {}, "jsonp").always(function(resp) {
    var countryCode = (resp && resp.country) ? resp.country : "";
    callback(countryCode);
  });
}
```
_Note that the callback must still be called in the event of an error, hence the use of `always` in this example._  
_Tip: store the result in a cookie to avoid repeat lookups!_

**initialCountry**  
Type: `String` Default: `""`  
Set the initial country selection by specifying it's country code. You can also set it to `"auto"`, which will lookup the user's country based on their IP address (requires the `geoIpLookup` option - [see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/default-country-ip.html)). Note that the `"auto"` option will not update the country selection if the input already contains a number.

If you leave `initialCountry` blank, it will default to the first country in the list.

**nationalMode**  
Type: `Boolean` Default: `true`  
Allow users to enter national numbers (and not have to think about international dial codes). Formatting, validation and placeholders still work. Then you can use `getNumber` to extract a full international number - [see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/national-mode.html). This option now defaults to `true`, and it is recommended that you leave it that way as it provides a better experience for the user.

**numberType**  
Type: `String` Default: `"MOBILE"`  
Specify one of the keys from the global enum `intlTelInputUtils.numberType` e.g. `"FIXED_LINE"` to tell the plugin you're expecting that type of number. Currently this is only used to set the placeholder to the right type of number.

**onlyCountries**  
Type: `Array` Default: `undefined`  
Display only the countries you specify - [see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/only-countries-europe.html).

**preferredCountries**  
Type: `Array` Default: `["us", "gb"]`  
Specify the countries to appear at the top of the list.

**~~preventInvalidNumbers~~ [REMOVED]**  
Prevent the user from entering invalid characters. Unfortunately this had to be removed for the reasons listed here: [#79 Limit Input Characters to Formatted String Length](https://github.com/jackocnr/intl-tel-input/issues/79#issuecomment-121799307).

**separateDialCode**  
Type: `Boolean` Default: `false`  
Display the country dial code next to the selected flag so it's not part of the typed number. Note that this will disable `nationalMode` because technically we are dealing with international numbers, but with the dial code separated.

**utilsScript**  
Type: `String` Default: `""` Example: `"build/js/utils.js"`  
Enable formatting/validation etc. by specifying the path to the included utils.js script (also available from [cdnjs.com](https://cdnjs.com/libraries/intl-tel-input)), which is fetched only when the page has finished loading (on window.load) to prevent blocking. When instantiating the plugin, we return a [deferred object](https://api.jquery.com/category/deferred-object/), so you can use `.done(callback)` to know when initialisation requests like this have finished. See [Utilities Script](#utilities-script) for more information. _Note that if you're lazy loading the plugin script itself (intlTelInput.js) this will not work and you will need to use the `loadUtils` method instead._


## Public Methods
**destroy**  
Remove the plugin from the input, and unbind any event listeners.  
```js
$("#phone").intlTelInput("destroy");
```

**getExtension**  
Get the extension from the current number. Requires the `utilsScript` option.
```js
var extension = $("#phone").intlTelInput("getExtension");
```
Returns a string e.g. if the input value was `"(702) 555-5555 ext. 1234"`, this would return `"1234"`

**getNumber**  
Get the current number in the given format (defaults to [E.164 standard](http://en.wikipedia.org/wiki/E.164)). The different formats are available in the enum `intlTelInputUtils.numberFormat` - which you can see [here](https://github.com/jackocnr/intl-tel-input/blob/master/src/js/utils.js#L109). Requires the `utilsScript` option. _Note that even if `nationalMode` is enabled, this can still return a full international number. Also note that this method expects a valid number, and so should only be used after validation._  
```js
var intlNumber = $("#phone").intlTelInput("getNumber");
// or
var ntlNumber = $("#phone").intlTelInput("getNumber", intlTelInputUtils.numberFormat.E164);
```
Returns a string e.g. `"+17024181234"`

**getNumberType**  
Get the type (fixed-line/mobile/toll-free etc) of the current number. Requires the `utilsScript` option.  
```js
var numberType = $("#phone").intlTelInput("getNumberType");
```
Returns an integer, which you can match against the [various options](https://github.com/googlei18n/libphonenumber/blob/master/javascript/i18n/phonenumbers/phonenumberutil.js#L896) in the global enum `intlTelInputUtils.numberType` e.g.  
```js
if (numberType == intlTelInputUtils.numberType.MOBILE) {
    // is a mobile number
}
```
_Note that in the US there's no way to differentiate between fixed-line and mobile numbers, so instead it will return `FIXED_LINE_OR_MOBILE`._

**getSelectedCountryData**  
Get the country data for the currently selected flag.  
```js
var countryData = $("#phone").intlTelInput("getSelectedCountryData");
```
Returns something like this:
```js
{
  name: "Afghanistan (‫افغانستان‬‎)",
  iso2: "af",
  dialCode: "93"
}
```

**getValidationError**  
Get more information about a validation error. Requires the `utilsScript` option.  
```js
var error = $("#phone").intlTelInput("getValidationError");
```
Returns an integer, which you can match against the [various options](https://github.com/jackocnr/intl-tel-input/blob/master/src/js/utils.js#L153) in the global enum `intlTelInputUtils.validationError` e.g.  
```js
if (error == intlTelInputUtils.validationError.TOO_SHORT) {
    // the number is too short
}
```

**isValidNumber**  
Validate the current number - [see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/is-valid-number.html). Expects an internationally formatted number (unless `nationalMode` is enabled). If validation fails, you can use `getValidationError` to get more information. Requires the `utilsScript` option. Also see `getNumberType` if you want to make sure the user enters a certain type of number e.g. a mobile number.  
```js
var isValid = $("#phone").intlTelInput("isValidNumber");
```
Returns: `true`/`false`

**setCountry**  
Change the country selection (e.g. when the user is entering their address).  
```js
$("#phone").intlTelInput("setCountry", "gb");
```

**setNumber**  
Insert a number, and update the selected flag accordingly. _Note that by default, if `nationalMode` is enabled it will try to use national formatting._  
```js
$("#phone").intlTelInput("setNumber", "+447733123456");
```


## Static Methods

**getCountryData**  
Get all of the plugin's country data - either to re-use elsewhere e.g. to populate a country dropdown - [see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/country-sync.html), or to modify - [see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/modify-country-data.html). Note that any modifications must be done before initialising the plugin.  
```js
var countryData = $.fn.intlTelInput.getCountryData();
```
Returns an array of country objects:
```js
[{
  name: "Afghanistan (‫افغانستان‬‎)",
  iso2: "af",
  dialCode: "93"
}, ...]
```

**loadUtils**  
_Note: this is only needed if you're lazy loading the plugin script itself (intlTelInput.js). If not then just use the `utilsScript` option._  
Load the utils.js script (included in the lib directory) to enable formatting/validation etc. See [Utilities Script](#utilities-script) for more information.
```js
$.fn.intlTelInput.loadUtils("build/js/utils.js");
```

**~~setCountryData~~ [REMOVED]**  
Set the plugin's country data. This method was removed because it makes much more sense to just use `getCountryData` and then modify that ([see example](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/modify-country-data.html)) instead of having to generate the whole thing yourself - the country data has become increasingly complicated and for each country we now have five properties: the name, [iso2 country code](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2), international dial code, priority (in case two countries have the same international dial code), and finally a list of area codes used in that country - see [data.js](https://github.com/jackocnr/intl-tel-input/blob/master/src/js/data.js#L36) for more info.


## Events
You can listen for the following events on the input.

**countrychange**  
This is triggered when the user selects a country from the dropdown.
```js
$("#phone").on("countrychange", function(e, countryData) {
  // do something with countryData
});
```
See an example here: [Country sync](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/country-sync.html)


## Utilities Script
A custom build of Google's [libphonenumber](http://libphonenumber.googlecode.com) which enables the following features:

* Formatting upon initialisation, as well as with `getNumber` and `setNumber`
* Validation with `isValidNumber`, `getNumberType` and `getValidationError` methods
* Placeholder set to an example number for the selected country - even specify the type of number (e.g. mobile) using the `numberType` option
* Extract the standardised (E.164) international number with `getNumber` even when using the `nationalMode` option

International number formatting/validation is hard (it varies by country/district, and we currently support ~230 countries). The only comprehensive solution I have found is libphonenumber, which I have precompiled into a single JavaScript file and included in the lib directory. Unfortunately even after minification it is still ~215KB, but if you use the `utilsScript` option then it will only fetch the script when the page has finished loading (to prevent blocking).

To recompile [Utilities Script](#utilities-script) see js-docs in top of [utils.js](src/js/utils.js).


## Troubleshooting
**Submitting the full international number when in nationalMode**  
If you're submitting the form using Ajax, simply use `getNumber` to get the number before sending it. If you're using the standard form POST method, you have two options. The easiest thing to do is simply update the input value using `getNumber` in a submit handler:  
```js
$("form").submit(function() {
  myInput.val(myInput.intlTelInput("getNumber"));
});
```
But this way the user will see their value change when they submit the form, which is weird. A better solution would be to update the value of a separate hidden input, and then read that POST variable on the server instead. See an example of this solution [here](http://jackocnr.com/node_modules/intl-tel-input/examples/gen/hidden-input.html).  

**Full width input**  
If you want your input to be full-width, you need to set the container to be the same i.e.
```css
.intl-tel-input {width: 100%;}
```

**Input margin**  
For the sake of alignment, the default CSS forces the input's vertical margin to `0px`. If you want vertical margin, you should add it to the container (with class `intl-tel-input`).

**Displaying error messages**  
If your error handling code inserts an error message before the `<input>` it will break the layout. Instead you must insert it before the container (with class `intl-tel-input`).

**Dropdown position**  
The dropdown should automatically appear above/below the input depending on the available space. For this to work properly, you must only initialise the plugin after the `<input>` has been added to the DOM.

**Placeholders**  
In order to get the automatic country-specific placeholders, simply omit the placeholder attribute on the `<input>`.

**Bootstrap input groups**  
A couple of CSS fixes are required to get the plugin to play nice with Bootstrap [input groups](http://getbootstrap.com/components/#input-groups). You can see a Codepen [here](http://codepen.io/jackocnr/pen/EyPXed).  
_Note: there is currently [a bug](https://bugs.webkit.org/show_bug.cgi?id=141822) in Mobile Safari which causes a crash when you click the dropdown arrow (a CSS triangle) inside an input group. The simplest workaround is to remove the CSS triangle with this line: `.intl-tel-input .iti-flag .arrow {border: none;}`_


## Contributing
See the [contributing guide](https://github.com/jackocnr/intl-tel-input/blob/master/.github/CONTRIBUTING.md).


## Attributions
* Flag images from [region-flags](https://github.com/behdad/region-flags)
* Original country data from mledoze's [World countries in JSON, CSV and XML](https://github.com/mledoze/countries)
* Formatting/validation/example number code from [libphonenumber](http://libphonenumber.googlecode.com)
* Feature contributions are listed in the wiki: [Contributions](https://github.com/jackocnr/intl-tel-input/wiki/Contributions)


## Links
* List of [sites using intl-tel-input](https://github.com/jackocnr/intl-tel-input/wiki/Sites-using-intl-tel-input)
* List of [integrations with intl-tel-input](https://github.com/jackocnr/intl-tel-input/wiki/Integrations)
* Android native port: [IntlPhoneInput](https://github.com/Rimoto/IntlPhoneInput)
* Typescript type definitions are available in the [DefinitelyTyped repo](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/intl-tel-input/intl-tel-input.d.ts) (more info [here](https://github.com/jackocnr/intl-tel-input/issues/433#issuecomment-228517623))
