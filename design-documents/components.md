# Framework components

1. Every framework component is a separate composer package with type `magento2-library` and name prefix `magento2/library-`
2. Every `\Magento\Framework\App` component is a separate package.
2. Classes that used to live in `Magento\Framework` and `Magento\Framework\App` namespaces are moved to root folders of corresponding components, while all other classes of those components are moved into folder with the name of component. Root namespace for such components is configured to be `\Magento\Framework` and `\Magento\Framework\App` to avoid renaming classes.

*Before*
```
Magento/
  Framework/
    App/
      Action/
        AbstractAction.php
        ...
      ActionInterface.php
```
*Now*
```
Magento/
  Framework/
    App/
      Action/
        Action/
          AbstractAction.php
        ActionInterface.php
        composer.json
```

3. Every framework component can specify `/etc` directory where application configuration can be provided.
