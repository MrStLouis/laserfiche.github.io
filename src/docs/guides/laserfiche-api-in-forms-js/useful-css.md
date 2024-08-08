---
layout: default
title: Useful CSS
nav_order: 4
has_children: false
parent: Using the Laserfiche API from Forms JS
grand_parent: Guides
---

<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

# Basic New CSS
Notable classes:
-	`condensed-table`
	-	Squishes the table field label and search/filter options into one row
-	`hidden`
	-	`display: none;` Hides the field on rendered forms, displays them with a hidden label in the designer
-	`invisible`
	-	`visibility: hidden;` Hides the field on rendered forms, displays them with a hidden label in the designer

Notable features:
-	Bootstrap style focus field color and errors.
-	Lengthens all fields to take up the full width of the container (dates/time fields)
-	Highlights individual rows within a table, and adds a hover color effect

See it in action:  
![Records Management Searches](./assets/records-management-search-form.gif){: width="500px"}

{: .note }
**Hint:** Click on the copy icon in the top right corner of the code block to copy the code to your clipboard.

```css
/* Generic Form CSS */
.form-container {
  --read-only-bg-color: rgba(0, 0, 0, 0);
  --read-only-border: none;
  --primary-border-color: #ced4da;
  --primary-shadow-color-focus: #80bdff;
  --simple-input-height: calc(1.5em + 0.75rem + 2px);
  --simple-input-padding: 0.375rem 0.75rem;
  --error-border-color: #b91818;
  --error-shadow-color-focus: rgba(220, 53, 69, 0.25);
  /* Make input errors more obvious */
  --error-input-bg-url: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' fill='none' stroke='%23dc3545' viewBox='0 0 12 12'%3e%3ccircle cx='6' cy='6' r='4.5'/%3e%3cpath stroke-linejoin='round' d='M5.8 3.6h.4L6 6.5z'/%3e%3ccircle cx='6' cy='8.2' r='.6' fill='%23dc3545' stroke='none'/%3e%3c/svg%3e");
  /* --error-input-bg-url: none; */
}
/* Hidden fields show in the designer */
.pdc.hidden:has(.selectors-display) {
  display: unset !important;
  border: 1px dashed gray;
  border-radius: 0.25rem;
}
.pdc.hidden:not(:has(.selectors-display)) {
  display: none;
}
.pdc.invisible:has(.selectors-display) {
  visibility: visible !important;
  border: 1px dashed gray;
  border-radius: 0.25rem;
}
.pdc.invisible:not(:has(.selectors-display)) {
  visibility: hidden;
  opacity: 0;
}
.pdc.invisible:has(.selectors-display)::after,
.pdc.hidden:has(.selectors-display)::after {
  content: 'hidden';
  position: absolute;
  right: 0;
  top: 0;
  background: lightgray;
  border-radius: 0.25rem;
  padding: 0.25rem;
  margin: 2px;
}
.pdc {
  transition:
    visibility 0.3s linear,
    opacity 0.3s linear;
}
.pdc.selected {
  border-radius: 0.25rem;
}
/* Set fields to have full width/height/modern borders by default */
fl-date-time.fl-component div.cf-field .fl-datetime-inputs {
  max-width: 100%;
  display: flex;
}
fl-date-time.fl-component
  div.cf-field
  .fl-datetime-inputs
  > .fl-date-container {
  flex: 1;
  max-width: unset;
}
fl-date-time.fl-component
  div.cf-field
  .fl-datetime-inputs
  > .fl-time-container {
  flex: 0.5;
  max-width: unset;
}
fl-date-time .flatpickr-wrapper {
  width: 100%;
}
fl-date-time.fl-component div.cf-field .fl-datetime-inputs .lib-datetime-picker input {
  border-radius: 0.25rem;
  border: 1px solid var(--primary-border-color);
  -webkit-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -moz-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -o-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  height: var(--simple-input-height);
  padding: var(--simple-input-padding);
}
fl-date-time.fl-component div.cf-field .fl-datetime-inputs input:focus {
  border-color: var(--primary-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--primary-shadow-color-focus);
}
fl-date-time.fl-component
  div.cf-field
  .fl-datetime-inputs
  input.ng-wrapper-invalid.ng-touched {
  border-color: var(--error-border-color);
  padding-right: calc(1.5em + 0.75rem) !important;
  background-image: var(--error-input-bg-url);
  background-repeat: no-repeat;
  background-position: right calc(0.375em + 0.1875rem) center;
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}
fl-date-time.fl-component
  div.cf-field
  .fl-datetime-inputs
  input.ng-wrapper-invalid.ng-touched:focus {
  border-color: var(--error-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--error-shadow-color-focus);
}
fl-address.fl-component div.cf-field input {
  border-radius: 0.25rem;
  border: 1px solid var(--primary-border-color);
  -webkit-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -moz-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -o-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  height: var(--simple-input-height);
  padding: var(--simple-input-padding);
}
fl-address.fl-component div.cf-field input:focus {
  border-color: var(--primary-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--primary-shadow-color-focus);
}
fl-address.fl-component div.cf-field input.ng-invalid.ng-touched {
  border-color: var(--error-border-color);
  padding-right: calc(1.5em + 0.75rem) !important;
  background-image: var(--error-input-bg-url);
  background-repeat: no-repeat;
  background-position: right calc(0.375em + 0.1875rem) center;
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}
fl-address.fl-component div.cf-field input.ng-invalid.ng-touched:focus {
  border-color: var(--error-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--error-shadow-color-focus);
}
/* Highlight errored fields more */
/* Uncomment to make error field backgrounds more prominent
.fl-component.required:has(> div div.error-message) {
  background-color: lightpink;
  border-radius: 10px;
  } 
*/
.fl-component input:not([type='checkbox']):not([type='radio'])[readonly],
.fl-component textarea[readonly] {
  border: var(--read-only-border) !important;
  background: var(--read-only-bg-color) !important;
  resize: none;
}
fl-single-line.fl-component div.cf-field input[type='text'],
fl-multi-line.fl-component div.cf-field textarea,
fl-email.fl-component div.cf-field input[type='email'],
fl-number.fl-component div.cf-field .number-inputs > input {
  border-radius: 0.25rem;
  border: 1px solid var(--primary-border-color);
  -webkit-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -moz-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -o-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  height: var(--simple-input-height);
  padding: var(--simple-input-padding);
}
fl-single-line.fl-component div.cf-field input[type='text']:focus,
fl-multi-line.fl-component div.cf-field textarea:focus,
fl-email.fl-component div.cf-field input[type='email']:focus,
fl-number.fl-component div.cf-field .number-inputs > input:focus {
  border-color: var(--primary-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--primary-shadow-color-focus);
}
fl-single-line.fl-component
  div.cf-field
  input[type='text'].ng-invalid.ng-touched,
fl-multi-line.fl-component div.cf-field textarea.ng-invalid.ng-touched,
fl-email.fl-component div.cf-field input[type='email'].ng-invalid.ng-touched,
fl-number.fl-component
  div.cf-field
  .number-inputs
  > input.ng-invalid.ng-touched {
  border-color: var(--error-border-color);
  padding-right: calc(1.5em + 0.75rem) !important;
  background-image: var(--error-input-bg-url);
  background-repeat: no-repeat;
  background-position: right calc(0.375em + 0.1875rem) center;
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}
fl-single-line.fl-component
  div.cf-field
  input[type='text'].ng-invalid.ng-touched:focus,
fl-multi-line.fl-component div.cf-field textarea.ng-invalid.ng-touched:focus,
fl-email.fl-component
  div.cf-field
  input[type='email'].ng-invalid.ng-touched:focus,
fl-number.fl-component
  div.cf-field
  .number-inputs
  > input.ng-invalid.ng-touched:focus {
  border-color: var(--error-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--error-shadow-color-focus);
}
fl-rich-text.fl-component div.cf-field div.note-frame.card {
  border-radius: 0.25rem;
  border: 1px solid var(--primary-border-color);
  -webkit-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -moz-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -o-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  margin: 0.2rem;
}
fl-rich-text.fl-component div.cf-field div.note-frame.card:focus-within {
  border-color: var(--primary-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--primary-shadow-color-focus);
}
fl-rich-text.fl-component div.cf-field.hasError div.note-frame.card {
  border-color: var(--error-border-color);
}
fl-rich-text.fl-component
  div.cf-field.hasError
  div.note-frame.card:focus-within {
  border-color: var(--error-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--error-shadow-color-focus);
}
/*#region dropdown/checkbox/radio */
fl-dropdown.fl-component {
  div.cf-field ng-select > .ng-select-container {
    height: var(--simple-input-height) !important;
  }
  div.cf-field ng-select > .ng-select-container input {
    height: var(--simple-input-height);
    padding: var(--simple-input-padding);
  }
}
fl-dropdown.fl-component,
fl-checkbox.fl-component,
fl-radio.fl-component {
  div.cf-field ng-select > .ng-select-container:not(.ng-has-value) {
    height: calc(var(--simple-input-height) + 3.5px) !important;
  }
  div.cf-field ng-select > .ng-select-container {
    border-radius: 0.25rem;
    border: 1px solid var(--primary-border-color);
    -webkit-transition:
      border-color 0.15s ease-in-out,
      box-shadow 0.15s ease-in-out;
    -moz-transition:
      border-color 0.15s ease-in-out,
      box-shadow 0.15s ease-in-out;
    -o-transition:
      border-color 0.15s ease-in-out,
      box-shadow 0.15s ease-in-out;
    transition:
      border-color 0.15s ease-in-out,
      box-shadow 0.15s ease-in-out;
    /* padding: var(--simple-input-padding); */
  }

  div.cf-field ng-select.ng-select-focused > .ng-select-container {
    border-color: var(--primary-border-color);
    outline: 0;
    box-shadow: 0 0 0 0.2rem var(--primary-shadow-color-focus);
  }
  div.cf-field ng-select.ng-invalid.ng-touched > .ng-select-container {
    border-color: var(--error-border-color);
    padding-right: calc(0.75em + 2.3125rem) !important;
    background-image: var(--error-input-bg-url);
    background-repeat: no-repeat;
    background-position:
      right 2.75rem center,
      right 1.75rem center;
    background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
  }
  div.cf-field
    ng-select.ng-invalid.ng-touched.ng-select-focused
    > .ng-select-container {
    border-color: var(--error-border-color);
    outline: 0;
    box-shadow: 0 0 0 0.2rem var(--error-shadow-color-focus);
  }
}
/* #endregion */
/**
  * Highlight table/collection search fields more
  */
.fl-table.fl-component
  .repeatables-pagination-top
  .repeatables-pagination-functions
  .repeatables-pagination-search-wrapper
  .repeatables-pagination-search,
.fl-collection.fl-component
  .repeatables-pagination-top
  .repeatables-pagination-functions
  .repeatables-pagination-search-wrapper
  .repeatables-pagination-search {
  border-radius: 0.25rem;
  border: 1px solid var(--primary-border-color);
  -webkit-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -moz-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  -o-transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
  height: var(--simple-input-height);
  padding: var(--simple-input-padding);
}
.fl-table.fl-component
  .repeatables-pagination-top
  .repeatables-pagination-functions
  .repeatables-pagination-search-wrapper
  .repeatables-pagination-search:focus,
.fl-collection.fl-component
  .repeatables-pagination-top
  .repeatables-pagination-functions
  .repeatables-pagination-search-wrapper
  .repeatables-pagination-search:focus {
  border-color: var(--primary-border-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem var(--primary-shadow-color-focus);
}
/* Make tables more readable */
@media screen and (min-width: 600px) {
  fl-form .form-container .form-header {
    display: flex;
    align-items: flex-end;
    flex-direction: row;
  }
  .form-pagination {
    flex: 1;
    padding-left: 20px;
  }
  .form-pagination > nav > a.form-tab,
  .form-pagination > nav > a.form-tab.active {
    font-size: 16px;
  }
  .condensed-table > fl-table > div {
    display: flex;
    flex-wrap: wrap;
  }
  .condensed-table .section-title {
    flex: 1;
    padding: 0px 20px;
    display: flex;
    align-items: center;
  }
  .condensed-table .section-title label {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin: 0px;
  }
  .condensed-table fl-repeatables-pagination:first-of-type {
    width: inherit;
  }
  .condensed-table fl-repeatables-pagination > .repeatables-pagination-top {
    margin: 8px 10px 0px 0px;
  }
  .condensed-table fl-repeatables-pagination > .repeatables-pagination-bottom {
    padding: 8px 10px;
  }
  .condensed-table
    fl-repeatables-pagination
    > .repeatables-pagination-bottom
    input.curPage {
    height: inherit;
  }
  .condensed-table th div.label-container {
    display: flex;
    justify-content: flex-start;
  }
  .fl-table.fl-component th.table-header-cell .field-label {
    text-align: inherit;
    padding-left: 10px;
  }
  .fl-table.fl-component th.table-header-cell {
    text-align: left;
    padding: 14px;
    border: none;
    border-bottom: 2px solid #ddd;
  }
  .fl-table.fl-component td.pdc {
    border: none;
    border-top: 1px solid #ddd;
    border-bottom: 1px solid #ddd;
  }
  .fl-table.fl-component tbody tr:nth-of-type(odd) {
    background-color: rgba(0, 0, 0, 0.05);
  }
  .fl-table.fl-component tbody tr {
    transition: background 0.2s ease-in;
  }
  .fl-table.fl-component tbody tr:hover {
    background-color: rgba(184, 218, 255, 0.5);
  }
}

```