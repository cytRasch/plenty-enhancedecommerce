# Tutorial for Google Enhanced e-Commerce Tracking in plentymarkets
### Version 1.0
To implement (almost) full functional enhanced e-commerce tracking, go through the following steps.

### Warning
Never do any changes in your running production enviroment. It is highly recommended to setup a new layout for this guide and please backup
your current active layout.

### Requirements
  - first, be sure to have UniversalAnalytics (analytics.js) running
  - the "new" fully inidvidual checkout is required
  -  you need a running plentymarkets online shop, of course
  - **No Google Tag Manager**

## Google Analytics setup
1. You need to activate the **Enhanced E-Commerce Setting** as you can see in the image below
![setup enhanced ecommerce](https://raw.githubusercontent.com/cytRasch/plenty-enhancedecommerce/master/img/ga-01.png)
<br>
<br>
<br>
<br>
2. After activation you nee to setup your checkout-matching funnels like this
![setup enhanced ecommerce](https://raw.githubusercontent.com/cytRasch/plenty-enhancedecommerce/master/img/ga-02.png)

**Be careful if you PayPal Plus, for example. In this case you have to split Payment & Shipping**\
plentymarkets manual for this: [https://www.plentymarkets.eu/handbuch/payment/paypal-plus/](https://www.plentymarkets.eu/handbuch/payment/paypal-plus/)

## plentymarkets setup
### 1. $PageDesignContent
1. Okay, now copy & paste your js-Tracking snippet from Google Analytics somewhere inside the `head`-Tag.
2. Add some needed plugins from Google Analytics that your code looks similar to this:
```html
<!-- ga -->
<script type="text/javascript">
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
	(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
	m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
	})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
	
    // ga init
	ga('create', 'UA-12345678', 'your-shop.de');
    
    // ga plugins
	ga('require', 'linkid');
	ga('require', 'ec');
	ga('set', 'forceSSL', true);
	ga('set', 'anonymizeIp', true);
	ga('set', '&cu', 'EUR');
</script>
```

3. Now we have to integrate the `pageview`-snippet to send all needed data to google. please add the following snippet (if possible) right before the `</body>`-Tag
```html
<script type="text/javascript">
    ga('send','pageview');
</script>
```
---
#### 1.1 $ItemViewCategoriesList
This is getting a little bit more complex. Here you can handle all activities like:
* addImpression
* addProduct
* and setAction

1. Right after the `$FormOpenOrder`-Tag in your for-loop you have to copy & paste this:
```html
<!-- ga -->
<script type="text/javascript">
    {% 
        # dynamic function naming
        $_fnName = 'onProductClick'.$ID;
    %}
    
    // send impression/view of products
    ga('ec:addImpression', {
        'id': '$ID',
        // send the clean name without any html-tags
        'name': '{% trim(strip_tags($Name[1])) %}',
        'category': '$CurrentCategoryName',
        'list': 'CategoryView',
        'position': $RowCount
    });
    
    // create dynamic function for adding products
    function $_fnName() {
        ga('ec:addProduct', {
            'id': '$ID',
            'name': '{% trim(strip_tags($Name[1])) %}',
            'category': '$CurrentCategoryName',
            'brand': '$Producer',
            'position': $RowCount,
            'price': '{% number_format($Price, 2, '.', '') %}'
        });
        ga('ec:setAction', 'click', {list: 'CategoryView'});
        
        // if you click on a product, say it to google
        ga('send', 'event', 'CategoryView', 'click', 'Results', {
            hitCallback: function() {
                document.location = '{% Link_Item($ID) %}';
            }
        });
    }
    
    // the document ready stuff (jQuery required)
    $(document).ready(function() {
        $('#identifier-$ID .PlentyWebshopButton').on('click',function(event) {
            ga('ec:addProduct', {
                'id': '$ID',
                'name': '{% trim(strip_tags($Name[1])) %}',
                'category': '$CurrentCategoryName',
                'brand': '$Producer',
                'price': '{% number_format($Price, 2, '.', '') %}',
                // this is for qty = 1, if you have a qty input on your
                // category list you have to add more functionality
                'quantity': 1
            });
            ga('ec:setAction', 'add');
            ga('send', 'event', 'CategoryView', 'click', 'Add to Basket');
        });
    });
</script>
```

2. Now we have to add some functions to the product if the user interacts
* add a unique identifier to the product-wrapping html-tag, for example:
`id="identifier-$ID"`
```html
<!-- item box -->
<li class="col-xs-12 col-sm-3 col-md-4 col-lg-3 margin-bottom-2 center itemBox tileView onHover action-$ActionId" data-plenty-id="$ID" id="identifier-$ID">
	$FormOpenOrder
	...
	$FormCloseOrder
</li>
```
* Find all `<a>`-Tags with a link to the product page and add a `onclick`-function referenced to our generated function
```
<a class="name block" onclick="onProductClick$ID(); return !ga.loaded;" href="{% Link_Item($ID) %}">...</a>
```
* At last, find the button for adding your product to the basket and wrap it with an unique ID that it looks similar to this:
```html
<div class="buttonBox isAddToBasket" id="addToBasket-$ID">
	<button class="btn btn-primary text-center" data-plenty="click:Basket.addBasketItem(this)">
		<span class="glyphicon glyphicon-shopping-cart"></span>in den Warenkorb
	</button>
</div>
```
---
#### 1.2 $ItemViewSearchResultsList
This is almost the same procedure as you did with $ItemViewCategoriesList. Just change your reference from **CategoryView** to **SearchResult**

---
#### 1.3 $ItemViewSingleItem
1. At the very beginning simply add this short snippet:
```html
<!-- ga -->
<script type="text/javascript">
    ga('ec:addProduct', {
        'id': '$ID',
        'name': '{% trim(strip_tags($Name[1])) %}',
        'category': '$CurrentCategoryName',
        'brand': '$Producer',
        'price': '{% number_format($Price, 2, '.', '') %}'
    });
    ga('ec:setAction', 'ItemView');
    
    $(document).ready(function(){
       
        $('.toBasketWrapper button').on('click',function(event) {
            ga("ec:addProduct", {
                "id": "$ID",
                'name': '{% trim(strip_tags($Name[1])) %}',
                "price": "{% number_format($Price, 2, '.', '') %}",
                "brand": "$Producer",
                "category": "$CurrentCategoryName",
                "position": 0,
                "quantity": $('.quantityInput').val()
            });
            ga("ec:setAction", "add");
            ga("send", "event", "ItemView", "click", "Add to Basket");
        });
        
    });
</script>
```

Well done, that's it. Now we can go to setup the checkout-process.
## 2. $PageDesignCheckout
Allright, this is getting a little bit more tricky. But if you want to track the checkout-process and if you want to see how your visitors interact, this is really important.

1. Paste your js-Tracking snippet with some additional plugins inside your `<head>`-Tag like this:
```html
<!-- ga -->
<script type="text/javascript">
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
	(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
	m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
	})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
	
	// ga init
	ga('create', 'UA-12345678', 'your-shop.de');
	
	// needed plugins
	ga('require', 'linkid');
    	ga('require', 'ec');
    	ga('set', 'forceSSL', true);
	ga('set', 'anonymizeIp', true);
	ga('set', '&cu', 'EUR');
	
	// recommended
	ga('require','displayfeatures');
	
	// send initial pageview
	ga('send','pageview');
</script>
```
---
### 2.1 Basket (Warenkorb)
1. Go to your ***category page!!*** called "Warenkorb" or something like this (not the Container `CheckoutBasketItemsList`)
2. Add the following snipper somewhere **after** `{% Container_CheckoutBasketItemsList() %}`:
```html
{# iterator #}
{% $_bN = 0 %}
<!-- ga -->
<script type="text/javascript">
	{% for $_b in GetCheckoutBasketItemsList() %}
		ga("ec:addProduct", {
			"id": "$_b->BasketItemID",
			"name": '{% trim(strip_tags($_b->BasketItemName[1])) %}',
			"price": "{% number_format($_b->BasketItemPrice, 2, '.', '') %}",
			"brand": "$_b->BasketItemProducerName",
			// this might be not the best solution
			// but currently there is no other
			// choose your level by yourself
			"category": "$_b->BasketItemCategoryName[Level3]",
			"position": $_bN,
			"quantity": $_b->BasketItemQuantity
		});
		
		{% $_bN = $_bN + 1 %}
		
	    $(document).ready(function(){
	        $('.rem-$_b->BasketItemID').on('click',function(event) {
	            ga("ec:addProduct", {
	                "id": "$_b->BasketItemID",
	                "name": '{% trim(strip_tags($_b->BasketItemName[1])) %}',
	                "price": "{% number_format($_b->BasketItemPrice, 2, '.', '') %}",
	                "brand": "$Producer",
	                "category": "$CurrentCategory",
	                "position": $_bN,
	                "quantity": $('#quantity-input-$_b->BasketItemID').val()
	            });
	            ga("ec:setAction", "remove");
	            ga("send", "event", "Basket", "click", "Remove from Basket");
	        });
	        
	    });
	{% endfor %}
	
	// send the first step to ga
	ga("ec:setAction", "checkout", {
		"step": 1
	});
	
	// say ga, there is a new page view
	ga('send','pageview');
</script>
```
2. Now open the Container `CheckoutBasketItemsList` in the checkout-area of the _Webdesign_
3. Find the `<a>`-tag or `<button>`-tag (or whatever) with the attriebute `onclick="plenty.BasketService.removeItem($BasketItemID)"`
4. Add a new class with **rem-$BasketItemID** to this tag to get referenced to the correspondending event, like this:
```html
<a class="btn btn-primary onlyIcon va-middle rem-$BasketItemID" onclick="plenty.BasketService.removeItem($BasketItemID)"><span class="glyphicon glyphicon-trash" aria-hidden="true"></span><span class="sr-only">Artikel aus Warenkorb löschen</span></a>
```
---
### 2.2 Checkout (Kasse)
1. Add a new `data`-tag to all of your process tabs, for example **Bestelldetails** has to get `data-step="3"`
```html
{% if GetGlobal("ShowTabDetails") %}
<li class="small-12 medium-3 large-3 columns">
	<span id="checkoutTabOrderDetails" role="tab" aria-controls="checkoutPanelOrderDetails" data-step="3">
		<span class="akf-bubbles3" aria-hidden="true"></span>
		<span>Bestelldetails</span>
	</span>
</li>
{% endif %}
```
...all other steps like this with 4,5,6,7 and so on.
2. Now find the existing code-snippet and add some anaylitics stuff, so it looks almost (depending on your CMStool-version):
```html
<script type="text/javascript">
    // it is not possible to get the item cateory at the order-success page
    // so we have to save it on our own locally in the storage
    // (in version 6)
    var categoryArray = [];
	sessionStorage.removeItem("categories");
	
	{% $_bN = 0 %}
	{% for $_b in GetCheckoutBasketItemsList() %}
		ga("ec:addProduct", {
			"id": "$_b->BasketItemID",
			"name": '{% trim(strip_tags($_b->BasketItemName[1])) %}',
			"price": "{% number_format($_b->BasketItemPrice, 2, '.', '') %}",
			"brand": "$_b->BasketItemProducerName",
			"category": "$_b->BasketItemCategoryName[Level3]",
			"position": $_bN,
			"quantity": $_b->BasketItemQuantity
		});
		
		// save category
		categoryArray[$_bN] = "$_b->BasketItemCategoryName[Level3]"
		{% $_bN = $_bN + 1 %}
	{% endfor %}
	
	// put the categories into the session storage of the browser
	sessionStorage["categories"] = JSON.stringify(categoryArray);

	(function($) {
		$(document).ready(function() {
			// scroll to top on tab change
			plenty.NavigatorService.afterChange(function() {
				$( window ).trigger( 'contentChanged' );
				var distanceTop = 50;
				var navigationOffsetTop = $('.checkout-navigation').offset().top
				if ( navigationOffsetTop - distanceTop < $(document).scrollTop() ) {
					$('html, body').animate({ scrollTop: ( navigationOffsetTop - distanceTop ) }, 500);
				}
				
				// send the active steps to the analytics funnel
				ga("ec:setAction", "checkout", {
					"step": $('[data-plenty-checkout="navigation"] .active > span').attr('data-step')
				});
				
				// and the pageview
				ga('send','pageview');
					
				return true;
			});
		});
	}(jQuery));
</script>
```
---
### 2.3 CheckoutCoupon
Let's go the container for the coupon to save the coupon up to the end of the checkout process until the user gets the order confirmation.
To do this, just add this short snippet to the container:
1. If a coupon is active
```html
{% if $CouponHasActiveCoupon %}
...
<script type="text/javascript">
	// add coupon code to session storage
	sessionStorage.setItem("coupon", "$CouponActiveCouponCode");
</script>
{% else %}
...
{% endif %}
```
2. If no coupon is active or it has been removed while going through the checkout that in finally looks like
```html
{% if $CouponHasActiveCoupon %}
...
<script type="text/javascript">
	// add coupon code to session storage
	sessionStorage.setItem("coupon", "$CouponActiveCouponCode");
</script>
{% else %}
...
<script type="text/javascript">
	// remove coupon code from session storage
	sessionStorage.removeItem("coupon");
</script>
{% endif %}
````
---
### 2.4 Order confirmation (Bestellbestätigung)
In your categories section go to the container which is setup for your order confirmation and add these lines of code:
```html
{% $_confirm = GetCheckoutOrderConfirmation() %}
<!-- ga -->
<script type="text/javascript">
	// get the categories for each item from session storage
	var finalCategory = JSON.parse(sessionStorage['categories']);
    
	{% $_bN = 0 %}
    {% for $_p in GetCheckoutOrderConfirmationItemsList() %}
	    {% if $_p->OrderConfirmationItemID %}
	        ga("ec:addProduct", {
	    		"id": "$_p->OrderConfirmationItemID",
	    		"name": '{% trim(strip_tags($_p->OrderConfirmationItemName[1])) %}',
	    		"price": "{% number_format($_p->OrderConfirmationItemPriceTotal, 2, '.', '') %}",
	    		"brand": "$_p->OrderConfirmationItemProducerName",
	    		"category": finalCategory[$_bN],
	    		"quantity": $_p->OrderConfirmationItemQuantity
	        });
	    {% endif %}
	    {% $_bN = $_bN + 1 %}
    {% endfor %}
    
    // if coupon is set, get it
    if(sessionStorage.getItem("coupon") !== null) {
    	var cpVal = sessionStorage.getItem("coupon");
    } else {
    	var cpVal = '';
    }
    
    ga('ec:setAction', 'purchase',{
        'id': '$_confirm->OrderID',
        'affiliation': 'https://your-shop.de',
        'revenue': '{% number_format($_confirm->TotalAmount, 2, '.', '') %}',
        'tax': '{% number_format(($_confirm->TotalAmount - $_confirm->TotalAmountNet), 2, '.', '') %}',
        'shipping': '{% number_format($_confirm->ShippingAmount, 2, '.', '') %}',
        'coupon': cpVal
    });
    
    ga('send','pageview');
</script>    
```
---
And that's it. Now you can see how your customers behave and where you have to commit changes to offer a better experience or a better service to reduce breakups.

![track your user behavior in analytics](https://raw.githubusercontent.com/cytRasch/plenty-enhancedecommerce/master/img/ga-03.png)
