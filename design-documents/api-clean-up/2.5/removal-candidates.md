# Already deprecated interfaces that need to be removed in 2.5

Depending on the amount of the refactoring required, some of the interfaces will not be removed in 2.5.

1. app/code/Magento/CatalogInventory/Model/ResourceModel/QtyCounterInterface.php: Module is deprecated and should be removed in 2.5
1. app/code/Magento/CatalogInventory/Model/ResourceModel/StockStatusFilterInterface.php: Module is deprecated and should be removed in 2.5
1. app/code/Magento/CatalogInventory/Model/Spi/StockRegistryProviderInterface.php: Module is deprecated and should be removed in 2.5
1. app/code/Magento/CatalogInventory/Model/Spi/StockStateProviderInterface.php: Module is deprecated and should be removed in 2.5
1. app/code/Magento/CatalogInventory/Observer/ParentItemProcessorInterface.php: Module is deprecated and should be removed in 2.5
1. app/code/Magento/CatalogSearch/Model/ResourceModel/Fulltext/Collection/DefaultFilterStrategyApplyCheckerInterface.php: Deprecated since May 2019
1. app/code/Magento/MediaGalleryApi/Model/Asset/Command/DeleteByDirectoryPathInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/MediaGalleryApi/Model/Asset/Command/DeleteByPathInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/MediaGalleryApi/Model/Asset/Command/GetByIdInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/MediaGalleryApi/Model/Asset/Command/GetByPathInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/MediaGalleryApi/Model/Asset/Command/SaveInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/MediaGalleryApi/Model/Keyword/Command/GetAssetKeywordsInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/MediaGalleryApi/Model/Keyword/Command/SaveAssetKeywordsInterface.php: Can be removed, has been deprecated since April 2020
1. app/code/Magento/Payment/Model/Method/ConfigInterface.php: Deprecated since 2018
1. app/code/Magento/Search/Model/QueryFactoryInterface.php: Deprecated since 2017
1. app/code/Magento/Store/Api/StoreResolverInterface.php: Deprecated since June 2018, has replacement
1. lib/internal/Magento/Framework/App/Action/HttpHeadActionInterface.php: Deprecated since March 2019. Has replacement
1. lib/internal/Magento/Framework/App/TemplateTypesInterface.php: Deprecated since 2017
1. lib/internal/Magento/Framework/Filesystem/ExtendedDriverInterface.php: Implementation will be merged into \Magento\Framework\Filesystem\DriverInterface
1. lib/internal/Magento/Framework/Image/Adapter/UploadConfigInterface.php: Deprecated since 2018
1. lib/internal/Magento/Framework/MessageQueue/ConfigInterface.php: Deprecated since 2016
1. lib/internal/Magento/Framework/Module/Output/ConfigInterface.php: Deprecated since 2017
1. lib/internal/Magento/Framework/Option/ArrayInterface.php: Deprecated since 2018. Will require refactoring of usages
1. lib/internal/Magento/Framework/Session/SidResolverInterface.php: Deprecated since March 2020
1. lib/internal/Magento/Framework/View/Asset/Bundle/ConfigInterface.php: Deprecated since March 2017
1. lib/internal/Magento/Framework/View/Element/UiComponent/Config/ManagerInterface.php: Deprecated since Apr 2017
1. magento2ee/app/code/Magento/AdvancedCheckout/Model/IsProductInStockInterface.php: Already deprecated, since Apr 2020,  has replacement

# Interfaces to be marked as deprecated in 2.4 and removed in 2.5

Depending on the amount of the refactoring required, some of the interfaces will not be removed in 2.5.

1. app/code/Magento/Analytics/Model/ConfigInterface.php: should be replaced by Magento\Framework\Config\DataInterface
1. app/code/Magento/Analytics/Model/Connector/Http/ClientInterface.php: should be replaced by Psr\Http\Client\ClientInterface
1. app/code/Magento/Analytics/Model/Connector/Http/ResponseHandlerInterface.php: related to app/code/Magento/Analytics/Model/Connector/Http/ClientInterface.php	
1. app/code/Magento/Analytics/ReportXml/ConfigInterface.php: should be replaced by Magento\Framework\Config\DataInterface
1. app/code/Magento/GraphQl/Model/Query/ContextFactoryInterface.php: Interfaces are not necessary for factories, should be deprecated and later removed
1. app/code/Magento/Sales/Model/ResourceModel/HelperInterface.php: 1 usage, need to be eliminated in favor of private implementation
1. lib/internal/Magento/Framework/Filter/Encrypt/AdapterInterface.php: No implementations, there are preference but they implement zend interface directly. Seems unused
1. lib/internal/Magento/Framework/GraphQl/Query/PostFetchProcessorInterface.php: Not used
1. lib/internal/Magento/Framework/Model/Operation/ReadInterface.php: Not used
1. lib/internal/Magento/Framework/Model/Operation/WriteInterface.php: Not used
1. lib/internal/Magento/Framework/Model/ResourceModel/Db/ProcessEntityRelationInterface.php: Not used
1. lib/internal/Magento/Framework/Module/ResourceInterface.php: Implementation is deprecated with explanation why
1. lib/internal/Magento/Framework/View/Asset/SourceFileGeneratorInterface.php:	Not used
1. lib/internal/Magento/Framework/View/Design/Theme/Domain/PhysicalInterface.php: Not used
1. lib/internal/Magento/Framework/View/Design/Theme/Domain/StagingInterface.php: Not used
1. lib/internal/Magento/Framework/View/Design/Theme/Domain/VirtualInterface.php: Not used
1. app/code/Magento/CustomerGraphQl/Api/ValidateCustomerDataInterface.php: GraphQL modules should be extensible via GraphQL schema. They are not domain modules and must not contain Api folder
1. app/code/Magento/Catalog/Model/EntityInterface.php: Unused interface, exists for 5 years
