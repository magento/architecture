### Remove __sleep and __wakeup methods from our code
#### Why?
* Serializing objects requires classes to always consider serialization process
and it is easy for multiple developers working on the same class to forget about it.
* PHP serialization opens up our code to potential security issues (Remote code execution).
* During serialization objects of classes that do not expect serialization may be serialized
anyway causing unexpected behaviour.

#### Current situation
The only place we serialize objects (as far as I am aware of) is storing User and Acl objects
in admin session for convenience during admin auth process. Indirect serialization caused
us to add __sleep and __wakeup methods to ~33 classes.

#### What has to be done
##### Admin session
I propose creating 2 interfaces in the Magento\Backend\Spi namespace:
* SessionUserHydratorInterface
* SessionAclHydratorInterface
 
Both interfaces will have 2 methods:
* extract(User|Acl $user|$acl): array - which will extract given object's
data into an array of scalars
* hydrate(User|Acl $target, array $data): void - taking array of scalars
and filling given object with it
 
Only arrays of scalars will be stored in session. getUser, setUser, getAcl, setAcl methods
must be added to the Magento\Backend\Model\Auth\Session class to prevent serializing
objects into admin session.
 
##### CodeMess rule
Add a PHPMD rule which will be triggered by classes
having methods called __wakeup and __sleep
 
##### Existing code
Remove __sleep and __wakeup from our code
