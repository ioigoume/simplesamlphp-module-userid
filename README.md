# simplesamlphp-module-userid

A SimpleSAMLphp module containing authentication processing filters for
managing user identifier information based on the attributes received from
the Identity Provider (IdP).

## OpaqueSmartID

This filter is based on the `smartattributes:SmartID` authentication
processing filter included in the SimpleSAMLphp distribution. As such,
it can be used to provide consistent user identifiers when there are
multiple SAML IdPs releasing different identifier attributes.
The functionality of the original filter has been extended to support the
following identifier properties:

- **Global uniqueness**: This can be ensured by specifying a scope for the
  generated user identifier.
- **Opaqueness**: The generated user identifier (excluding the "@scope" portion)
  is based on the SHA-256 hash of the attributes received by the IdP, resulting
  in an opaque 64-character long string that by itself provides no information about
  the identified user.

To support these properties, the identifier is generated using the first
non-empty attribute from a given list of candidate attributes (`candidates`).
The filter can be configured to skip the generation process by
including/excluding certain IdPs. In this case, the persistent user identifier
value is copied (pass-through) from another list of candidate attributes
(`cuid_candidates`).

In both cases, at least one non-empty attribute is required, otherwise
authentication fails with an exception.

### Configuration

The following configuration options are available:

- `candidates`: An array of attributes names to consider for generating the
  user identifier attribute. Defaults to:
  - `eduPersonUniqueId`
  - `eduPersonPrincipalName`
  - `eduPersonTargetedID`
  - `openid`
  - `linkedin_targetedID`
  - `facebook_targetedID`
  - `windowslive_targetedID`
  - `twitter_targetedID`
- `authority_candidate_map`: An array of IdP entityIDs which are mapped to
  specific attributes names to consider for generating the user identifier.
  Defaults to the empty array. If no IdP in this map matches, then the user
  identifier is generated based on the attributes specified in `candidates`.
- `cuid_candidates`: An array of attributes names to consider as the user
  identifier attribute for whitelisted/blacklisted IdP tags. Defaults to:
  - `voPersonID`
  - `subject-id`
  - `eduPersonUniqueId`
- `id_attribute`. A string to use as the name of the newly added attribute.
  Defaults to `smart_id`.
- `add_authority`: A boolean to indicate whether or not to append the SAML
  AuthenticatingAuthority to the resulting identifier. This can be useful to
  indicate what SAML IdP was used, in case the original identifier is not
  scoped. Defaults to `true`.
- `add_candidate`: A boolean to indicate whether or not to prepend the
  candidate attribute name to the resulting identifier. This can be useful
  to indicate the attribute from which the identifier comes from. Defaults
  to `true`.
- `scope`: A string to use as the scope portion of the generated user
  identifier. There is no default scope value; however, you should consider
  scoping the generated attribute for creating globally unique identifiers
  that can be used across infrastructures.
- `set_userid_attribute`: A boolean to indicate whether or not to assign the
  generated user identifier to the `UserID` state parameter. Defaults to
  `true`. If this is set to `false`, SSP will attempt to use the value of the
  `eduPersonPrincipalName` attribute, leading to errors when the latter is
  not available.
- `skip_authority_list`: Optional, an array of IdP entityIDs that should be
  excluded from the authority part of the user id source.
- `idp_tag_whitelist`: Optional, an array of tags that the auth process
  should be executed
- `authority_map`: Optional, an array of IdP entityIDs that have been
  modified. The keys of the array are the new entityIDs and the values are the
  old ones.
- `idp_tag_blacklist`: Optional, an array of tags that the auth process
  should not be executed

The generated identifiers have the following form:

```bash
SHA-256(AttributeName:AttributeValue!AuthenticatingAuthority!SecretSalt)
```

or, if a scope has been specified:

```bash
SHA-256(AttributeName:AttributeValue!AuthenticatingAuthority!SecretSalt)@scope
```

### Example configuration

```php
authproc = [
    ...
    '60' => [
        'class' => 'uid:OpaqueSmartID',
        'candidates' => [
            'eduPersonUniqueId',
            'eduPersonPrincipalName',
            'eduPersonTargetedID',
        ],
        'cuid_candidates' => [
            'voPersonID',
            'subject-id',
            'eduPersonUniqueId',
        ],
        'id_attribute' => 'eduPersonUniqueId',
        'add_candidate' => false,
        'add_authority' => true,
        'scope' => 'example.org',
        'skip_authority_list' => [
            'https://www.example1.org',
            'https://www.example2.org',
        ],
        'idp_tag_whitelist' => [
            'tag1',
            'tag2',
        ],
    ],
```

## PersistentNameID2Attribute

The `userid:PersistentNameID2Attribute` is a SimpleSAMLphp authentication
processing filter for generating an attribute from the persistent NameID.

### Configuration

The following configuration options are available:

- `attribute`: Optional, a string to define the attribute name to save the
  NameID in. Defaults to `eduPersonTargetedID`
- `nameId`: Optional, a boolean to indicate whether or not to insert `NameID`
  attribute as a \SAML2\XML\saml\NameID object. Defaults to `true`.

### Example configuration

```php
authproc = [
    ...
    '61' => [
        'class' => 'userid:PersistentNameID2Attribute',
        'attribute' => 'eduPersonTargetedID',
        'nameId' => true,
    ],
```

## RequiredAttributes

The `userid:RequiredAttributes` is a SimpleSAMLphp authentication processing
filter for making attribute(s) mandatory. If the IdP doesn't release these
attributes then the authentication chain will stop with an error message
displayed in the UI.

### Configuration

The following configuration options are available:

- `attributes`: Optional, an array of attributes names which define the
  required attributes. Default values: givenName, sn, mail
- `custom_resolutions`: Optional, an array of entity IDs as keys and the custom
  error message as values . Defaults to empty array.

### Example configuration

```php
  authproc = [
      ...
      '62' => [
          'class' => 'userid:RequiredAttributes',
          'attributes' => [
              'givenName',
              'sn',
              'mail',
              'eduPersonScopedAffiliation',
          ],
          'custom_resolutions' => [
              'https://www.example1.org/' => 'Error message foo',
              'https://www.example2.org/' => 'Error message foo bar',
          ],
      ],
```

## Compatibility matrix

This table matches the module version with the supported SimpleSAMLphp version.

| Module | SimpleSAMLphp |
|:------:|:-------------:|
|  v1.x  |     v1.14     |
|  v2.x  |     v1.15     |
|  v3.x  |     v1.17     |

## License

Licensed under the Apache 2.0 license, for details see `LICENSE`.
