Inject service to Behat Context as Behat Step Argument Transformer argument
===========================================================================

The `Behat service autowiring feature <https://github.com/Behat/Behat/pull/1071>`_ allows to inject services from the configured service container to any of the `Step Argument Transformer <http://behat.org/en/latest/user_guide/context/definitions.html#step-argument-transformations>`_ method as argument. You can use this feature in combination with this extension as well. E.g.:

**Feature:**

.. code-block:: gherkin

  Feature: Magento and Behat DI connected
    As a developer
    In order to write Behat tests easily
    I should be able to inject services from the Magento DI into Behat Contexts

    Scenario: Injecting service from Magento DI to Behat Context as argument for Behat Step Paramater Transformation method
      Given A service has been successfully injected to the parameter transformation method while transforming "foobar"
      When I work with Behat
      Then I am happy

**Context:**

.. code-block:: php

    <?php

    use Behat\Behat\Context\Context;
    use Magento\Catalog\Api\Data\ProductInterface;
    use Magento\Catalog\Api\Data\ProductInterfaceFactory as ProductFactory;
    use Magento\Catalog\Api\ProductRepositoryInterface;

    class YourContext implements Context
    {
        /**
         * @Transform
         */
        public function transformStringToProduct(
            string $productSku,
            ProductRepositoryInterface $productRepository,
            ProductFactory $productFactory
        ): ProductInterface {
            Assert::assertInstanceOf(ProductRepositoryInterface::class, $productRepository);

            try {
                return $productRepository->get($productSku);
            } catch (NoSuchEntityException $e) {
                // product does not exsits - normally you would let the test fail here
                // but for this demonstration we will just create a new product in memory
                // also note that the product factory autogenerated even when it is requested from Behat
                return $productFactory->create()->setSku($productSku);
            }
        }

        /**
         * @Given A service has been successfully injected to the parameter transformation method while transforming :product
         */
        public function theProductSkuSuccessFullyTransformedToProduct(ProductInterface $product)
        {
            if (!$product instanceof ProductInterface) {
                throw new Exception('Something went wrong :(');
            }
        }
    }

**Configuration:**

.. code-block:: yaml

  default:
    suites:
      yoursuite:
        autowire: true
        
        contexts:
          - YourContext
        
        services: '@seec.magento2_extension.service_container'
