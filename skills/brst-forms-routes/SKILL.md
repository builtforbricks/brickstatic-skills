---
name: brst-forms-routes
description: "Use when a Bricks form or a BFB Advanced Form must keep working on a site exported with Brickstatic: choosing per form between This WordPress site and Another service, the form's Brickstatic group in the builder (brstExport* settings), static origins and the WordPress address for forms, Check forms, what each route needs and cannot do, and which other form elements the preflight lists as not handled."
---

# Brickstatic: where each form sends

Read `brst-start-here` first. On a static page there is no WordPress to answer a form. Each Bricks form on an exported
page chooses, in the builder, one of two destinations. There is never an endpoint hosted by Built for Bricks.

## The form's Brickstatic group

On Bricks' form element, Content tab, group **Brickstatic** (after Bricks' own form groups). Bricks keeps the
values in the element's settings, so they travel with the page and change through Bricks like any form setting.

| Label | Setting | Values |
|---|---|---|
| Destination | `brstExportRoute` | `origin`, "This WordPress site" (default) or `endpoint`, "Another service" |
| Endpoint URL | `brstExportEndpoint` | the service's address, http or https (Another service only) |
| Encoding | `brstExportEncoding` | `form`, "Form fields" (default) or `json`, "JSON" (Another service only); a form with a file field is always sent as form fields |
| Field names | `brstExportNames` | `simple`, "Simple keys (full_name)" from each label (default), or `labels`, "Labels as written" (Another service only). A field's own custom name, when set, is sent as is. |
| Success message | `brstExportSuccess` | empty: the form's own success message (Another service only) |
| Error message | `brstExportError` | default "Sorry, your message could not be sent. Please try again later." |
| My service verifies captcha tokens | `brstExportCaptcha` | keeps the captcha widget on the Another service route |

With nothing set, a form takes This WordPress site. Another service without a usable http(s) endpoint URL falls back
to This WordPress site, and the preflight warns (`forms_endpoint_missing`).

## This WordPress site (`origin`)

The static page posts to this WordPress install's `admin-ajax.php`, and Bricks runs the form exactly as it does on
the live site: its actions (email, webhook, newsletter, create post, redirect), the honeypot and the captcha check.

- **Needs** WordPress online and publicly reachable at `/wp-admin/admin-ajax.php`, at the **WordPress address for
  forms** (Settings, Address and forms, `brst_static_export.forms_origin_url`; this site by default; another address when the
  export is made from a staging copy). No password, access rule, redirect or IP allowlist on that address.
- **Needs** the static site's origin, so WordPress accepts the post: the **Base URL**'s origin counts on its own;
  other addresses go in **More addresses the forms accept** (static origins, `brst_static_export.static_origins`), one per line, exact origins such as
  `https://www.example.com` or one wildcard level such as `https://*.preview.example` for preview deploys. Only the two
  Bricks form actions answer those origins; nothing else on WordPress changes.
- **Needs** a re-export after the form's fields change.
- A redirect to a page of this site lands on the same page of the static site.
- Captcha keys are tied to host names: the static site's host name goes on the key's domain list.
- When WordPress cannot be reached, the form shows Bricks' error markup with the Error message, never a silent
  failure.

## Another service (`endpoint`)

The static page posts the fields to the Endpoint URL; WordPress is not involved and nothing is added to it.

- Field names are simple keys (`full_name`) or the labels as written; a field's own custom name wins. Encoding is
  form fields or JSON. JSON cannot carry files: a form with a file field is sent as form fields (multipart) with the
  file even when JSON is chosen. The builder says so under Encoding, and the preflight warns (`forms_json_file`).
- The honeypot is renamed `_gotcha`; a filled honeypot shows the success message and sends nothing.
- The form shows Bricks' own success and error markup and fires Bricks' form events. Without JavaScript it still
  posts to the endpoint as a plain form.
- **Runs only what the service does**: Bricks' form actions do not run. Spam filtering beyond the honeypot is the
  service's job, and the captcha widget stays only when **My service verifies captcha tokens** is ticked.
- `{{field}}` tokens in the success message are filled in the visitor's browser from the submitted values, as text
  (a token for a field that does not exist is left empty). Tokens that Bricks fills when the page renders, such as
  `{post_title}`, are filled at export time.

## BFB Advanced Forms

A form built with BFB Advanced Forms (BFB Elements Pro, element `bfbe-form`) always sends to this WordPress site, on
the same request as a Bricks form, so everything under This WordPress site applies (online WordPress, the static
origin, Check forms, a re-export after changes). Its Brickstatic group has the info line and **Error message**
(`brstExportError`) only; there is no Destination to set. Its uploads, promo codes and Save for later go to the
WordPress site's REST addresses (`/wp-json/bfbe/v1/...`), which stay absolute in the exported page.

- After a payment (Pay with Stripe), or a cancel, the visitor comes back to the page on the static site. A Success or
  Cancel address the designer typed is used as typed: a path such as `/thanks/` means the WordPress site
  (`forms_advanced_pay_address`); a full address on the static site is right. The "payment is being confirmed" line
  does not show on a static page (`forms_advanced_pay` suggests a thank-you page on the static site).
- A Save for later link opens the page on the static site and fills the answers back in.
- Cannot work on a static page: the WooCommerce cart button of a Total element (`forms_advanced_cart`) and actions that
  need a logged-in visitor: Update the user, Update post fields, Delete the post, a webhook for logged-in visitors
  only (`forms_advanced_login`).
- A form in quiet mode (a calculator that is never sent) is not a form for the export or the preflight.

Any other form element, and a form added with a shortcode, is not connected: the preflight lists it as
`forms_unhandled`. The fix is to rebuild it with Bricks' Form element or BFB Advanced Forms, or to exclude the page.

## Check forms

Under Destination, **Check forms** posts from WordPress to itself as each static origin and confirms the cross-origin
header, a fresh form nonce and the preflight answer for each. "Not checked ... the server could not reach itself" is
a host that cannot call itself during a request (often a one PHP worker host); it does not mean the form fails: test
it from the static site. Check forms does not post to services.

## Choosing, with the owner

1. Run the preflight; the `forms` row lists every form with its route.
2. For each form ask: must Bricks' actions run (the site's email, a webhook, a created post)? Then This WordPress
   site, and WordPress stays online. Is WordPress going away, or must the static site stand alone? Then Another
   service, with its endpoint URL from the owner.
3. Change the Brickstatic group through Bricks, on the owner's word. Then set the static origins under Settings, Address and forms
   for the WordPress-site forms, save, and run Check forms.
4. Re-export, then the owner submits each form once from the static site. Only that proves a route.

## Where a form can sit

Measured by the plugin's checks: a form in an AJAX popup (to this WordPress site) and in a static popup (to a
service); forms inside a query loop on both routes (each item's form keeps its own success text); a form inside a
component (to this WordPress site) and global-element forms on both routes (the preflight lists them too); file fields
on both routes (to this WordPress site, Bricks saves the file; to a service, the service gets it). BFB Advanced Forms
(1.3.2), from a static copy on another host name than WordPress: two steps with a field error, an upload, the send and
its email; Save for later and the resumed send; a Stripe payment against a stand-in (the return addresses on the static
site); a form inside a component (counted and rendered).

## Not measured

Real services and real captcha keys (the plugin's checks post to a local receiver); a service-route form in an AJAX
popup; popups inside a loop; a file field inside a loop or a component; more than one file per field; a service that
refuses multipart; a host's upload size limit; a BFB Advanced Form in a popup or a query loop, its promo codes and
quote PDF from the static site, real Stripe, PayPal or Mollie. Say so when a form is one of these.
