# CiviCRM Member Page (Drupal 11)

Adds a **Membership** tab to every user profile (`/user/{uid}/membership`) and a
**My membership** link in the account menu (`/my-membership`).

The page shows:

- name, primary address, email and phone from the linked CiviCRM contact
- any extra core or custom fields you configure
- all memberships (type, status, member since, valid until)
- links to edit address, phone and email, plus an optional renewal link

Members see only their own page. Staff with the permission
*View any user's membership page* can open anyone's page, and they get a
link to the contact in CiviCRM instead of the member edit links.

Requirements: Drupal 10.3+/11 and CiviCRM 5.60+ (APIv4 `address_primary` /
`email_primary` / `phone_primary` joins). FormBuilder (Afform) is needed for the edit forms.

---

## 1. Install

```bash
cp -r civi_member_page web/modules/custom/
# Edit config/install/civi_member_page.settings.yml first if you like (see step 3)
drush en civi_member_page -y
drush cr
```

## 2. Create the edit forms in CiviCRM FormBuilder

The edit links point to small FormBuilder forms that load and save the
**logged-in user's own contact**. Enable the extensions first:
*Administer → System Settings → Extensions*: **FormBuilder** and **Form Core**.

Then go to *Administer → Customize Data and Screens → Forms* and create three forms
(**+ New Form → Custom Form**). For each one:

1. Add an **Individual** entity. In its settings set **Autofill → Current User**,
   and turn off **Allow creating new** so it can only update.
2. Add the fields:
   - *My address*: Address block (street address, postal code, city, country).
     Set the location type to match what you use (usually *Home*), and keep *Primary*.
   - *My phone*: Phone block (phone, phone type).
   - *My email*: Email block (email).
3. On the **Form Settings** tab:
   - **Placement / Page route**: `civicrm/my-address`, `civicrm/my-phone`, `civicrm/my-email`
   - **Permission**: *Authenticated user* (or a role you choose)
   - **Post-submit redirect**: `/my-membership`
4. Save.

The paths match the defaults in the module config. If you use other URLs, update
the config (step 3). You can also combine all three into one form and point all links
at it, or set a link to `''` to hide it.

> Using classic CiviCRM profiles instead? A link like
> `/civicrm/profile/edit?gid=12&reset=1` also edits the current user's contact.

## 3. Configure fields and links

Settings live in `civi_member_page.settings`. Edit them with
`drush config:edit civi_member_page.settings`, or before installing in
`config/install/civi_member_page.settings.yml`.

```yaml
extra_fields:
  - birth_date
  - Member_Info.Member_Number      # custom field: CustomGroupName.FieldName
  - Member_Info.Section:label      # :label shows the option label for selects

links:
  address: '/civicrm/my-address'
  phone: '/civicrm/my-phone'
  email: '/civicrm/my-email'
  renew: '/civicrm/contribute/transact?reset=1&id=1'   # optional
```

To find the machine names of custom fields, use the CiviCRM API4 Explorer
(*Support → Developer → API4 Explorer*). Pick *Contact → getFields* and look at `name`.

Run `drush cr` after changing the config.

## 4. Customise the look

Copy `templates/civi-member-page.html.twig` into your theme's `templates/`
folder and edit it.

Page styling is in `css/member-page.css`.

## Notes

- **Security:** CiviCRM data is read with permission checks off. Access is controlled
  by this module instead: a user can only reach the contact linked to their own
  Drupal account through CiviCRM's UFMatch table, unless they have the staff permission.
- **Caching:** the page is never cached. CiviCRM changes appear immediately.
- **No linked contact:** CiviCRM normally creates one when a user logs in. Otherwise
  the page shows a short "contact the office" message.
