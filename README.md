# datastore
A simple key value data store to be used as simple backend for scripts, small web apps and similar applications
## Design
### Authentication
Authentication is done using either oAuth2 via Github or using a github personal access token.
If using oAuth2 login, team memberships are synchronized every hour but can be rotated by renewing the session cookie or by the instance admin incrementing the cookie version.
If using personal access token authentication the token should only be used to acquire a cookie as requests using personal access tokens directly are heavily rate limited. Team memberships are synchronized during cookie creation or renewal.

### Namespacing and Permissions
All Operations use a key that lives in a namespace that follows the following rules
- each key is owned by an owner, an owner is the only person that can change permissions and always has access to the value under it
- each key can have a permission object attached
- a permission object is a list of groups or username that specifies which operations are allowed for each entry
- if a key doesn't have a permission object it inherits it's parents permissions (determined by splitting at '/')
- the root namespace give read-write permission to the owner of the namespace (and thus all keys below it inherit these permission if they are not overwritten)
- owners can also attach validation javascript functions to a permission object
- /g/$group_name -> namespace owned by group called $group_name
- /u/$user_name -> namespace owned by user called $user_name

### Values
The design of the handling of operations on values is borrowed from redis, even though the underlying data store is database-backed not memory-backed.  
The following types exist (with some corresponding operations):
- string (limited to a maximum size of 10MB)
  - string manipulation
  - bitmaps
  - counters
- json (like string, but only allows valid json to be stored)
  - json patch, extract etc
  - json helpers like:
    - set
    - sorted sets
    - lists
- streams (mainly used for event streaming)
  streams are optionally size limited arrays indexed by timestamp based IDs
  clients can insert at the end of a stream and fetch all entries since a given timestamp
  clients can also fetch using a server-sent-events based approach to receive updates instantly
  they are mainly to be used for event management
