# woocommerce_maximum_discount
How to create a customize Maximum Discount on Woocommerce Coupon

I initially thought that this function would be integrated into WooCommerce soon. However, it appears that this hasn't happened, as evidenced by numerous developers searching for this query on Stack Overflow.

Consider this example: Coupon code "HELLO30" offers a 30% discount with a maximum saving limit of $40. Ordinarily, with a cart total of $1000, the discount would be $300 (30% of $1000). However, the following code ensures that the discount is capped at $40.

Now, let me guide you through the process.

To begin, open your theme's functions.php file to add the new field "maximum_discount."

```
// Add a custom field to Admin coupon settings pages
add_action('woocommerce_coupon_options', 'add_coupon_text_field', 10);
function add_coupon_text_field()
{
    woocommerce_wp_text_input(array(
        'id'                => 'maximum_discount',
        'label'             => __('Maximum discount', 'woocommerce'),
        'placeholder'       => '',
        'description'       => __('Maximum discount that the user can take?', 'woocommerce'),
        'desc_tip'    => true,
    ));
}

add_action('woocommerce_coupon_options_save', 'save_coupon_text_field', 10, 2);
function save_coupon_text_field($post_id, $coupon)
{
    if (isset($_POST['maximum_discount'])) {
        $coupon->update_meta_data('maximum_discount', sanitize_text_field($_POST['maximum_discount']));
        $coupon->save();
    }
}

```

Once you've saved the changes, you'll notice a new field under Marketing -> Coupons.


![image](https://github.com/dieptang/woocommerce_maximum_discount/assets/7411097/5c0ae188-4c45-4683-8351-fda56f9302b8)


Next, proceed by adding the following lines:


```
add_action('woocommerce_order_after_calculate_totals', [$this, "yoursite_order_after_calculate_totals"], 100, 2);
function yoursite_order_after_calculate_totals($taxes, $order)
	{
		if (!empty($order_coupons = $order->get_coupons())) {
			$total_discount = 0;

			$order_discount_total = $order->get_discount_total();
			$order_discount_total_backup = $order_discount_total; //take a backup for comparing

			foreach ($order_coupons as $key => $order_coupon) //loop through the order coupons
			{

				$coupon_code = $order_coupon->get_code();
				$coupon_id   = wc_get_coupon_id_by_code($coupon_code);

				if (0 === $coupon_id) //coupon not exists
				{
					continue;
				}

				$coupon = new WC_Coupon($coupon_id); // to perform is percentage type check 

				$max_discount = get_post_meta($coupon_id, 'maximum_discount', true);
				$coupon_discount = $order_coupon->get_discount();

				if (!empty($max_discount) && $max_discount < $coupon_discount) //max restriction enabled.
				{
					$order_discount_total = min($coupon_discount, $max_discount); 
				}
			}


			if ($order_discount_total_backup > $order_discount_total) // An extra amount is found. So need to update
			{
				$order->set_discount_total($order_discount_total); //set order discount total

				$order_total = $order->get_total();
				$order->set_total($order_total + ($order_discount_total_backup - $order_discount_total)); //set order total
			}
		}
		$order->save();
	}
```

There you have it! It seamlessly functions with the REST API as well. Best of luck with your implementation!


Author: 

Diep Tang
https://flane.vn
https://nanotravel.vm
https://giamgiaz.com
