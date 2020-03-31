# gmail-filters

This playbook builds a mailFilters.xml file that can be imported into the gmail web client.  It takes a filters.yml file as configuration.

To start, you will need to create the parent labels under `GMail -> Settings -> Labels -> Create new label
`:

* Customers
* Mailing Lists
* Partners
* People

Then, run the playbook:

```
ansible-playbook gen.yml
```

If you are using my encrypted filters, you will want to perform the following:

```
echo 'super-secret-vault-password' > .vault-password
```

Then, run the playbook:

```
ansible-playbook --vault-password-file .vault-password gen.xml
```

Either way, it should output `mailFilters.xml` which can then be imported into the GMail web client.

It's best to delete all existing filters and import for all changes, though you can select individual filters to import during the process.

Please note that if you apply to existing conversations, be prepared for a long process.  It's best to ensure your labeling structure is correct, then delete all filters, reimporting them with the apply to existing conversations option selected.

# filters.yml

Yes, included are my filters.  Yes, you can ask for the vault password.  If you don't want to use mine, create your own `filters.yml` file based on the example below.  The playbook will use the first found in this order:

* `filters.yml`
* `encrypted-filters.yml`

## Customers

Customers are identified using their email domains and the filter effectively looks like:

```
(to:*@domain1.com OR from:*@domain1.com) OR ...
```

## Mailing Lists

Mailing lists get created using their list ids, and have the option to skip the inbox.  Mailing list filters are unique in that they are two phased:

* Apply label
* Optionally archive them

To filter `list-name@domain.com`, the filter effectively looks like:

```
(list(list-name.domain.com))
```

If you do not want a mailing list to be archived, ensure to set `skip_inbox='false'`.  The mailing list archive filter is generated LAST to ensure proper filtering.  It also excludes emails sent to: cc: or bcc: addresses specified in the list variable: `mailing_list_archive_ignore_emails_to` as shown in the example below.

## Partners

Similar to customers, partners are labels using their email domains, and the filter is effectively the same.

## People

People can have multiple e-mail addresses, though filter on e-mails FROM them.

## Ignore Chats

Google Chat sends e-mails that might need filtering.  By default, `filter_chats` is enabled.  This should also happen AFTER the people get processed to ensure proper labeling.

# Example filters.yml

For readability, the generated xml and filters will be sorted inside of each list during generation.  It's only easier to maintain them in alphabetical order for your purposes.

```
---

# Filter chats -- archive immediately
filter_chats: yes

# Name of the parent label for customers -- more of a convenience
customers_parent: Customers

# The list of customers to create
customers:

  - label: "{{ customers_parent }}/Acme Inc"
    # do not immediately archive the e-mail
    skip_inbox: 'false'
    domains:
      - acme.com

  - label: "{{ customers_parent }}/Foo Inc"
    skip_inbox: 'false'
    # they have multiple domains
    domains:
      - foo.com
      - foobar.com

# maximum number of mailing lists for each archive rule -- this is important
# as it helps avoid the 1000 character limit for filters
mailing_list_archive_max_per_filter: 5

# generally, mailing lists will be filtered, unless they are to: cc: bcc: any
# address listed here
mailing_list_archive_ignore_emails_to:
  - jpreston@redhat.com
  - mrjoshuap@redhat.com

# Name of the parent label for mailing lists -- more of a convenience
mailing_lists_parent: Mailing Lists

# This is the list of mailing lists
mailing_lists:

  - label: "{{ mailing_lists_parent }}/Acme Updates"
    # if skip_inbox is 'true' then this list will be included in the archive step
    skip_inbox: 'true'
    lists:
      - updates.acme.com

  - label: "{{ mailing_lists_parent }}/Multiple Lists"
    skip_inbox: 'true'
    lists:
      - random.list.example.com
      - some.other.list.example.com
      - yet.another.list.example.com

  - label: "{{ mailing_lists_parent }}/Foo Alerts"
    # do not include this list in the archival step
    skip_inbox: 'false'
    lists:
      - alerts.foo.com

# Name of the parent label for partners -- more of a convenience
partners_parent: Partners

# the list of partners
partners:

  - label: "{{ partners_parent }}/Alice Tech"
    skip_inbox: 'false'
    domains:
      - alicetech.com

  - label: "{{ partners_parent }}/Bob Services"
    skip_inbox: 'false'
    domains:
      - bobsvcs.com

  - label: "{{ partners_parent }}/Charlie Group"
    skip_inbox: 'false'
    domains:
      - charliegroup.com
      - charlieg.com

# Name of the parent label for people -- more of a convenience
people_parent: People

# this list of people
people:

  - label: "{{ people_parent }}/Jane Doe"
    skip_inbox: 'false'
    emails:
      - jadoe@example.com
      - jane.doe@example.com

  - label: "{{ people_parent }}/John Doe"
    skip_inbox: 'false'
    emails:
      - jdoe@example.com
      - john.doe@example.com

  # Make an arbitrary filter and use an arbitrary label
  - label: System Notifications
    skip_inbox: 'true'
    emails:
      - system@notifications.com
      - root@notifications.com
```
