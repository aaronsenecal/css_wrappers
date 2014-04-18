# CSS Wrappers
Control the HTML markup surrounding the content of your Drupal site with simple, CSS-like selectors.


## The core concept
Inspired by [workflow tools](http://emmet.io/) and [template languages](http://jade-lang.com/), CSS Wrappers was created to promote better HTML markup while making the process of building and managing markup for different data types and view modes by allowing site builders and administrators to easily define "wrappers" for their content. The approach it takes to accomplish this goal is two-fold:

### A simple, flexible management interface for the markup surrounding data in varying contexts
Site administrators can define the markup surrounding their data using CSS selectors to describe a nested structure of HTML tags and attributes. For example:

- The string `div.field-wrapper` defines a wrapper consisting of a `div` with a `class` of `field-wrapper`.
- The string `dd.description.list-item[role="presentation"] > dl#nested-list` defines a wrapper consisting of a `dl` with an `id` of `nested-list`, within a `dd` with a `class` of `description list-item` and a `role` of `presentation`

### A pluggable, extensible inheritance system
Developers can define and alter the rules for determining where the data-surrounding markup in the given context is coming from. For example:

- This module defines handlers for an entity's Fields. It defines default wrappers that surround the Field, its label, its content, and each of the items therein, and establishes an inheritance chain that checks for wrapper markup defined for the given Field at the parent entity's default view mode if none is defined for its current view mode.
- A submodule defines wrappers for [Field Groups](https://drupal.org/project/field_group) in a similar fashion, and then alters the inheritance chain for Fields to check for Field wrapper markup defined in a containing Field Group. This allows administrators and builders of sites using the Field Group module to significantly reduce the amount of configuration necessary to, say, designate an ordered or unordered list of content contained in any number of fields by creating a Field Group that defines list markup for its contained fields, and simply dragging the desired fields into that group.
- Each of these modules also defines options that allow for even further customization of wrapper markup and behavior.

## How to use CSS Wrappers on your site
Add the CSS Wrappers module to your `sites/*/modules` directory, then enable it (and the included submodule, if desired) in the `admin/modules` administration interface. The module ships with functionality for Field wrappers, and an optional submodule adding functionality for Field Group wrappers. Both of these are managed through the field display UI:

- These wrappers can be edited by opening the Formatter Settings form for a Field or Field Group and editing values in the CSS Wrappers fieldset. This module, by default, support wrapper definitions for the entire field, the field label, the group of items comprising the field's content, and the field items themselves. The included submodule extends these wrapper definitions to Field Groups and, additionally, provides wrappers for an entire Field Group, its label, and its content.
- The wrapper definitions for a given field, and from where those definitions are inherited are available at a glance in the Field/Field Group's Formatter Settings summary.

## How to extend CSS Wrappers with additional wrapper types or inheritance rules
The functionality of this module can be extended both by defining wrapper settings, or altering settings that have been defined by other modules.

### Defining new wrappers
The wrapper settings defined and implemented by your module should consist of the following components:

- Parameters for one or more **Wrapper Groups** that establish the base settings and inheritance behavior for a group of wrapper elements, and consist of:
    - An **Inheritance Chain** that maps out the protocol for applying wrapper settings to content being presented in varying contexts, from the most specific (e. g. settings applied directly to an element) to the most generic (the system defaults).
    - **Additional Options** for the group that provide control over special settings, supplemental to the wrapper's selector.
    - One or more **Wrapper Types** that target specific portions sections of the content being wrapped (e. g. field labels or content).
- The implementation of those wrappers during display, typically using the various hooks invoked for the targeted content during `drupal_render()`.

At the core of the CSS Wrappers API is an associative array, defined in your module's implementation of `hook_css_wrapper_info()`, keyed with the machine names of each wrapper group being defined, and populated with arrays of the following settings for those groups:

- **label:** The human-readable name of the wrapper group, presented to administrators during editing.
- **options:** An associative array of options for the wrapper group, keyed with each option's machine name, and populated with a form API array that defines the option's default value, and how it should be presented for editing.
- **inheritance:** An associative array keyed with the names of testing function hooks for each inheritance level (implemented via `css_wrappers_inherit_HOOK()`), and populated with human-readable labels describing each inheritance level for administrative display. This array represents the inheritance chain for the wrapper group, with each testing function being executed in the order defined in the array when the settings for a wrapper are inherited.
- **types:** An associative array keyed with the machine name of each wrapper type within a group and populated with arrays defining the setting for each type, consisting of:
    - **label:** The human-readable name of the wrapper type, displayed to administrators during editing.
    - **selector:** The default CSS selector string for the wrapper type.
    - **options:** An array of options for the selector type, formatted similarly to the options array for the wrapper group. If absent, the options from the wrapper group are inherited.

### Altering existing wrappers
During the definition of wrapper groups in `css_wrapper_info()`, several alter hooks are included so that the behavior of wrappers defined by other modules can be extended and changed after they are defined:

- `hook_css_wrapper_group_info_alter()` and `hook_css_wrapper_group_info_GROUPNAME_alter()` expose an entire group definition for alteration. The former is invoked for each wrapper group, and the latter is invoked once for each group name matching `GROUPNAME`.
- `hook_css_wrapper_HOOK_alter()` and `hook_css_wrapper_HOOK_GROUPNAME_alter()`, are invoked for each of the keys at the base of a group's definition array (i. e. `options`, `inheritance`, then `types`) as `HOOK`, for each group in the case in the former, and groups matching `GROUPNAME` in the case of the latter.

## What's next?
This module is, at this point, not ideal for use in production sites. The [issue queue](https://gitlab.cws.oregonstate.edu/aaron.senecal/css-wrappers/issues) outlines several important performance, user-interface and workflow enhancements that should be completed before CSS Wrappers is production-ready.