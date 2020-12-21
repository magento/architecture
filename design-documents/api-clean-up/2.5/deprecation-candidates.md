# Interfaces that should be deprecated in 2.5

1. app/code/Magento/Catalog/Model/Layer/ContextInterface.php: Note from Kril: As Magento moves from inheritance-based APIs all such classes will be deprecated together with their corresponding abstract classes.
1. app/code/Magento/Catalog/Model/Product/Gallery/ImagesConfigFactoryInterface.php: No need to introduce interface for factories. Factory class must marked as @api instead
1. lib/internal/Magento/Framework/EntityManager/EntityMetadataInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/HydratorInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/MapperInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/AttributeInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/CheckIfExistsInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/CreateInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/DeleteInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/ExtensionInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/ReadInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/Operation/UpdateInterface.php: This component was never finished and not recommended for usage
1. lib/internal/Magento/Framework/EntityManager/OperationInterface.php: This component was never finished and not recommended for usage
