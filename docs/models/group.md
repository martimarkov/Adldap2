# The Group Model

## Getting a groups members

When you receive a `Group` model instance, it will contain a `member`
attribute which contains the distinguished names of all
the members inside the group.

```php
$group = $provider->search()->groups()->first();

foreach ($group->members as $member) {

    // 'cn=John Doe,dc=corp,dc=acme,dc=org'
    echo $member;

}
```

But this might not be useful, since we might actually want the models for each member.

This can be easily done with the `getMembers()` method on the group.

```php
$group = $provider->search()->groups()->first();

foreach ($group->getMembers() as $member) {

    // Will be an instance of a Adldap `Model`
    $member->getCommonName();

}
```

You should be aware however, that calling the `getMembers()` method will
query your `AD` server for **every** member contained in
the group to retrieve its model.

Think of this example below as what is being called behind the scenes:

```php
$group = $provider->search()->groups()->first();

foreach ($group->members as $member) {

    $model = $provider->search()->findByDn($member);

}
```

### Paginating Group members

The group you're looking for might contain hundreds / thousands of members.

In this case, your server might only return you a portion of the groups members.

To get around this limit, you need to ask your server to paginate the groups members through a select:

```php
$group = $provider->search()->groups()->select('member;range=0-500')->first();

foreach ($group->members as $member) {
    // We'll only have 500 members in this query.
}
```

Now, when we have the group instance, we'll only have the first `500` members inside this group. However, calling the `getMembers()` method will automatically retrieve the rest of the members for you:

```php
$group = $provider->search()->groups()->select('member;range=0-500')->first();

foreach ($group->getMembers() as $member) {
    
    // Adldap will automatically retrieve the next 500 records until it's retrieved all records.
    
    $member->getCommonName();
    
}
```

## Getting only a groups member names

To retrieve only the names of the members contained in a group, call the `getMemberNames()` method:

```php
foreach ($group->getMemberNames() as $name) {

    // Returns 'John Doe' 
    echo $name;

}
```

> **Note**: This method does not query your server for each member to retrieve its name. It
> only parses the distinguished names from the groups `member` attribute. This means that
> if you have paginated group members, you will need to perform another query yourself
> to retrieve the rest of the member names (or just call the `getMembers()` method).

## Setting Group Members

To set members that are apart of the group, you can perform this in two ways:

> **Note**: Remember, this will remove **all** pre-existing members, and set the new given members on the group.

```php
$members = [
    'cn=John Doe,dc=corp,dc=acme,dc=org',
    'cn=Jane Doe,dc=corp,dc=acme,dc=org',
];

$group->setMembers($members);

$group->save();
```

Or manually:

```php
$group->member = [
    'cn=John Doe,dc=corp,dc=acme,dc=org',
    'cn=Jane Doe,dc=corp,dc=acme,dc=org',
];

$group->save();
```

## Adding One Member

To add a single member to a group, use the `addMember()` method:

> **Note**: You do not need to call the `save()` method after adding a
> member. It's automatically called so you can determine
> if the member was successfully added.

```php
// We can provide a model, or just a plain DN of the new member
$user = $provider->search()->users()->first();

if ($group->addMember($user)) {

    // User was successfully added to the group!

}

// Or

$user = 'cn=John Doe,dc=corp,dc=acme,dc=org';

if ($group->addMember($user)) {
    //
}
```

## Removing One Member

To remove a single member to a group, use the `removeMember()` method:

```php
// We can provide a model, or just a plain DN of the existing member
$group = $provider->search()->groups()->first();

$member = $group->getMembers()->first();

if ($group->removeMember($member)) {

    // Member was successfully removed from the group!

}

// Or

$user = 'cn=John Doe,dc=corp,dc=acme,dc=org';

if ($group->removeMember($user)) {
    //
}
```

## Removing All Members

To remove all members, use the `removeMembers()` method:

```php
if ($group->removeMembers()) {

    // All members were successfully removed!

}
```
