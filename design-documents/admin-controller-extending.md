### Rule for validation of controllers extending other controllers
The practice of extending controllers is not particulary popular in Magento
core but is wildly used by 3rd party developers. Therefore it would make sense
to establish rules regarding extending controllers to prevent developers from bad
practices resulting in unexpected behavior and bugs.

#### Controllers meant for backend extended by frontend controllers
Here's an example of what might happen, let's say we have a controllers for
administrators to save a review for a product:
```php
class ReviewSave extends AbstractAction
{
    ....
    
    public function execute()
    {
        $product = $this->productRepo->getById($this->getRequest()->getParam('product_id'));
        $review = $this->reviewFactory->create();
        $review->setData($request->getParam('review'));
        $review->setProductId($product->getId());
        $this->reviewRepository->save($review);
        $this->messageManager->addSuccess('Saved');
    }
}
```
 
And then a developer decides to let customers to save reviews as well and simply extends
the controller above for a new ReviewSave controller outside of Adminhtml namespace.
It will work fine for now, but what if the original controller changes?
 
Like admin-only functionality has been added?
```php
$review->setCreatedByAdmin(true);
```
Or a redirect to an admin page will be issued?
```php
$this->redirect('admin/reviews');
```

This situations will cause unexpected behavior for the controller meant for customers.

#### What can be done?
To avoid backward incompatible issues we cannot just make FrontController to check whether matched
action instance extends backend abstract action when _frontend_ area is accessed.
 
What we can do is to create a PHPStan rule that will check all ActionInterface implementations and
allow extending backend's AbstractAction only if an action has _Controller\Adminhtml_ in it's namespace.
