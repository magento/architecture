# Framework components

1. Every component is a separate composer package
2. Every `\Magento\Framework\App` component is a separate package.
2. For backward compatibility in GitHub repo classes that used to live in `Magento\Framework` and `Magento\Framework\App` namespaces are moved to root folders of corresponding components, while all other classes of those components are moved into folder with the name of component.

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
