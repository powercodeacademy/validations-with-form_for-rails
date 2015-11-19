# Validations with `form_for`

## Objectives

- use form_for to display a form with Validations
- print out full error messages above the form
- customize aspects of the text_field error message if possible

## Notes

Explain how form_for provides an abstraction for displaying a form aware of errors on inputs because of the model-based form. The form builder knows the object, knows the field, can automatically pre-fill and provide error class.

form_for makes adding basic validation information easy but you still need to be able to customize it and can always drop to text_field_tag and form_tag family of helpers to provide more granular control.
