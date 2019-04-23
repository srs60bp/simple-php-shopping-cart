# simple-php-shopping-cart
simple-php-shopping-cart
View demo: https://phppot.com/demo/simple-php-shopping-cart/
Download:  https://phppot.com/downloads/simple-php-shopping-cart.zip

Simple PHP Shopping Cart
Last modified on September 1st, 2018 by Vincy.

Building a PHP shopping cart eCommerce software is simple and easy. In this tutorial, let’s create a simple PHP shopping cart software with MySQL. This shopping cart application is purposely kept simple and as minimal as possible. You can download this and quickly customize it for your needs.

PHP Shopping Cart Software Development Overview

PHP shopping cart example has the following functionality: 

    Retrieve product information from the database.
    Create product gallery for the shopping cart.
    Manage cart items using the PHP session.
    Handle add, edit, remove and empty cart actions.

By retrieving the product results from the database, the details like product name, code, price and the product photos are returned in an array format. This array will be iterated to form the product gallery interface with the add-to-cart option. 

The added cart items are stored and managed in the PHP shopping cart session. The cart items will be cleared once the session is expired. Also, this code has an option to clear the entire cart or to remove any particular item from the cart, explicitly.

File Structure

The shopping cart software example has the following file structure. The file names and description about the actions performed with the files are listed here. So, you can simply build your own gallery-based shopping cart software with these files listed below.

    dbcontroller.php – the DAO operation like implementing connection and fetching data are performed with the file.
    index.php – the product gallery for the shopping cart is displayed on the landing page by using this PHP file.
    style.css – the cross-browser compatible minimal styles applied here to showcase products for the shopping cart.
    tblproduct.sql – contains SQL script with the product table structure and the data.
    product-images – a folder containing image file for the products to be shown in the gallery. 

Create Product Gallery for Shopping Cart

If you are familiar with online shopping then can able to understand the importance of the product gallery in a shopping cart application. Generally, people will not visit the online stores to directly purchase the products in their mind. We need to provide a catalog of the available products to let them have a glance with it. It leads to promote your products for the impulsive purchase sometime.

In the following code, I have shown the PHP script to get the product results from the database. The result will be an array and iterated to create product card in each iteration. The DBController.php class will handle the DAO operation to fetch products result from the database. I have created the DBController instance to run the SELECT query.

<?php
$product_array = $db_handle->runQuery("SELECT * FROM tblproduct ORDER BY id ASC");
if (!empty($product_array)) { 
	foreach($product_array as $key=>$value){
?>
	<div class="product-item">
		<form method="post" action="index.php?action=add&code=<?php echo $product_array[$key]["code"]; ?>">
		<div class="product-image"><img src="<?php echo $product_array[$key]["image"]; ?>"></div>
		<div class="product-tile-footer">
		<div class="product-title"><?php echo $product_array[$key]["name"]; ?></div>
		<div class="product-price"><?php echo "$".$product_array[$key]["price"]; ?></div>
		<div class="cart-action"><input type="text" class="product-quantity" name="quantity" value="1" size="2" /><input type="submit" value="Add to Cart" class="btnAddAction" /></div>
		</div>
		</form>
	</div>
<?php
	}
}
?>

Adding Products  to Shopping Cart

After creating the product gallery page, we need to work on the PHP code to perform the cart actions like add-to-cart, remove a single item from the cart, clear the cart and more. In the above code, I have added the HTML option to add the product to the shopping cart from the product gallery. On clicking the Add to Cart button from the product card, the product id will be submitted to the PHP via a HTML form.

In PHP, I have received and process the cart action with a switch control statement. I have created separate cases in the PHP switch to handle the add-to-cart, remove-single, empty-cart actions. The action is passed to the argument for the switch statement to execute a corresponding action case as requested.

The  ‘Add to cart’ action will be handled with the “add” case. In this case, I received the product id and the quantity. The cart session is created to store the submitted product in it. If the selected product is already added in the session, then the quantity will be updated with the reference of the PHP session index. 

case "add":
	if(!empty($_POST["quantity"])) {
		$productByCode = $db_handle->runQuery("SELECT * FROM tblproduct WHERE code='" . $_GET["code"] . "'");
		$itemArray = array($productByCode[0]["code"]=>array('name'=>$productByCode[0]["name"], 'code'=>$productByCode[0]["code"], 'quantity'=>$_POST["quantity"], 'price'=>$productByCode[0]["price"], 'image'=>$productByCode[0]["image"]));
		
		if(!empty($_SESSION["cart_item"])) {
			if(in_array($productByCode[0]["code"],array_keys($_SESSION["cart_item"]))) {
				foreach($_SESSION["cart_item"] as $k => $v) {
						if($productByCode[0]["code"] == $k) {
							if(empty($_SESSION["cart_item"][$k]["quantity"])) {
								$_SESSION["cart_item"][$k]["quantity"] = 0;
							}
							$_SESSION["cart_item"][$k]["quantity"] += $_POST["quantity"];
						}
				}
			} else {
				$_SESSION["cart_item"] = array_merge($_SESSION["cart_item"],$itemArray);
			}
		} else {
			$_SESSION["cart_item"] = $itemArray;
		}
	}
	break;

List Cart Items from the PHP Session

This code shows the HTML to display the shopping cart with action controls. Here, the cart session is iterated to list the cart items in a tabular format. Each row shows the product image, title, price, item quantity with a remove option. Also, it shows the total item quantity and the total item price by summing up the individual cart items.

<div id="shopping-cart">
<div class="txt-heading">Shopping Cart</div>

<a id="btnEmpty" href="index.php?action=empty">Empty Cart</a>
<?php
if(isset($_SESSION["cart_item"])){
    $total_quantity = 0;
    $total_price = 0;
?>	
<table class="tbl-cart" cellpadding="10" cellspacing="1">
<tbody>
<tr>
<th style="text-align:left;">Name</th>
<th style="text-align:left;">Code</th>
<th style="text-align:right;" width="5%">Quantity</th>
<th style="text-align:right;" width="10%">Unit Price</th>
<th style="text-align:right;" width="10%">Price</th>
<th style="text-align:center;" width="5%">Remove</th>
</tr>	
<?php		
    foreach ($_SESSION["cart_item"] as $item){
        $item_price = $item["quantity"]*$item["price"];
		?>
				<tr>
				<td><img src="<?php echo $item["image"]; ?>" class="cart-item-image" /><?php echo $item["name"]; ?></td>
				<td><?php echo $item["code"]; ?></td>
				<td style="text-align:right;"><?php echo $item["quantity"]; ?></td>
				<td  style="text-align:right;"><?php echo "$ ".$item["price"]; ?></td>
				<td  style="text-align:right;"><?php echo "$ ". number_format($item_price,2); ?></td>
				<td style="text-align:center;"><a href="index.php?action=remove&code=<?php echo $item["code"]; ?>" class="btnRemoveAction"><img src="icon-delete.png" alt="Remove Item" /></a></td>
				</tr>
				<?php
				$total_quantity += $item["quantity"];
				$total_price += ($item["price"]*$item["quantity"]);
		}
		?>

<tr>
<td colspan="2" align="right">Total:</td>
<td align="right"><?php echo $total_quantity; ?></td>
<td align="right" colspan="2"><strong><?php echo "$ ".number_format($total_price, 2); ?></strong></td>
<td></td>
</tr>
</tbody>
</table>		
  <?php
} else {
?>
<div class="no-records">Your Cart is Empty</div>
<?php 
}
?>
</div>

Removing or Clearing Cart Item

The cart session array is iterated to display the cart item. Each cart item will have a remove link. If the shopping cart user clicks the remove link shown in each cart item, then it will be deleted from the cart session by executing. I have provided an option to wipe out the cart session. The ‘Empty Cart’button control above the shopping cart will be used to clear the cart completely.

The following PHP code shows the “remove” and the “empty” cases to handle the shopping cart remove/clear actions. I used PHP unset() to clear the cart session to delete the added item from the shopping cart.

case "remove":
	if(!empty($_SESSION["cart_item"])) {
		foreach($_SESSION["cart_item"] as $k => $v) {
			if($_GET["code"] == $k)
				unset($_SESSION["cart_item"][$k]);				
			if(empty($_SESSION["cart_item"]))
				unset($_SESSION["cart_item"]);
		}
	}
	break;
case "empty":
	unset($_SESSION["cart_item"]);
        break;

Database Product Table for Shopping Cart

Import the following SQL script by using your database client to create the product table and load data into it. The product data will be read and displayed in the product gallery.

--
-- Table structure for table `tblproduct`
--

CREATE TABLE `tblproduct` (
  `id` int(8) NOT NULL,
  `name` varchar(255) NOT NULL,
  `code` varchar(255) NOT NULL,
  `image` text NOT NULL,
  `price` double(10,2) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

--
-- Dumping data for table `tblproduct`
--

INSERT INTO `tblproduct` (`id`, `name`, `code`, `image`, `price`) VALUES
(1, 'FinePix Pro2 3D Camera', '3DcAM01', 'product-images/camera.jpg', 1500.00),
(2, 'EXP Portable Hard Drive', 'USB02', 'product-images/external-hard-drive.jpg', 800.00),
(3, 'Luxury Ultra thin Wrist Watch', 'wristWear03', 'product-images/watch.jpg', 300.00),
(4, 'XP 1155 Intel Core Laptop', 'LPN45', 'product-images/laptop.jpg', 800.00);

--
-- Indexes for table `tblproduct`
--
ALTER TABLE `tblproduct`
  ADD PRIMARY KEY (`id`),
  ADD UNIQUE KEY `product_code` (`code`);

--
-- AUTO_INCREMENT for dumped tables
--

--
-- AUTO_INCREMENT for table `tblproduct`
--
ALTER TABLE `tblproduct`
  MODIFY `id` int(8) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=5;
COMMIT;

Simple PHP Shopping Cart Output

Below screenshot shows the product gallery of this simple PHP shopping cart example. Also, the cart items are listed from the PHP session above the gallery.

php-shopping-cart

 
