<?php
/**
 * Plugin Name: BuddyPress WooCommerce Products
 * Description: Allows users to create WooCommerce products from their BuddyPress profiles.
 * Version: 1.0
 * Text Domain: buddypress-woocommerce-products
 * Domain Path: /languages
 */

// Exit if accessed directly
if (!defined('ABSPATH')) exit;

// Activation hook
function bp_wc_products_activate() {
    // Check if WooCommerce is activated
    if (!class_exists('WooCommerce')) {
        deactivate_plugins(plugin_basename(__FILE__));
        wp_die(__('This plugin requires WooCommerce to be activated!', 'buddypress-woocommerce-products'));
    }
}
register_activation_hook(__FILE__, 'bp_wc_products_activate');

// Deactivation hook
function bp_wc_products_deactivate() {
    // Placeholder for deactivation actions if needed
}
register_deactivation_hook(__FILE__, 'bp_wc_products_deactivate');

// Add a new tab to BuddyPress profiles
function bp_wc_products_setup_nav() {
    global $bp;

    bp_core_new_nav_item(array(
        'name' => __('My Products', 'buddypress-woocommerce-products'),
        'slug' => 'my-products',
        'screen_function' => 'bp_wc_products_screen',
        'position' => 40,
        'parent_url' => bp_loggedin_user_domain() . '/my-products/',
        'parent_slug' => $bp->profile->slug,
    ));
}
add_action('bp_setup_nav', 'bp_wc_products_setup_nav');

// Screen function for the "My Products" tab
function bp_wc_products_screen() {
    add_action('bp_template_content', 'bp_wc_products_screen_content');
    bp_core_load_template(apply_filters('bp_core_template_plugin', 'members/single/plugins'));
}

// Screen content function for the "My Products" tab
function bp_wc_products_screen_content() {
    // Check if form was submitted
    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
        // Handle form submission
        bp_wc_products_handle_submission();
    }

    // Display form
    echo '<form method="POST" enctype="multipart/form-data">';
    echo '<label for="product-title">' . __('Product Title', 'buddypress-woocommerce-products') . '</label><br>';
    echo '<input type="text" id="product-title" name="product-title"><br>';

    echo '<label for="product-description">' . __('Product Description', 'buddypress-woocommerce-products') . '</label><br>';
    echo '<textarea id="product-description" name="product-description"></textarea><br>';

    echo '<label for="product-price">' . __('Product Price', 'buddypress-woocommerce-products') . '</label><br>';
    echo '<input type="number" step="0.01" id="product-price" name="product-price"><br>';

    echo '<label for="product-category">' . __('Product Category', 'buddypress-woocommerce-products') . '</label><br>';
    echo '<select id="product-category" name="product-category">';
    
    // Get product categories
    $categories = get_terms('product_cat', array('hide_empty' => false));
    
    foreach ($categories as $category) {
        echo '<option value="' . $category->term_id . '">' . $category->name . '</option>';
    }
    
    echo '</select><br>';

    echo '<label for="product-image">' . __('Product Image', 'buddypress-woocommerce-products') . '</label><br>';
    echo '<input type="file" id="product-image" name="product-image"><br>';
        
   
    wp_nonce_field('bp_wc_products_create');

    echo '<button type="submit">' . __('Create Product', 'buddypress-woocommerce-products') . '</button>';
    echo '</form>';

    // Query for user's products
    $args = array(
        'post_type' => 'product',
        'post_status' => 'publish',
        'author' => get_current_user_id(),
    );
    $query = new WP_Query($args);

    // Display user's products
    if ($query->have_posts()) {
        echo '<h2>' . __('My Products', 'buddypress-woocommerce-products') . '</h2>';
        while ($query->have_posts()) {
            $query->the_post();
            echo '<div>';
            the_post_thumbnail();
            echo '<h3><a href="' . get_permalink() . '">' . get_the_title() . '</a></h3>';
            echo '<p>' . get_woocommerce_currency_symbol() . get_post_meta(get_the_ID(), '_price', true) . '</p>';
            echo '</div>';
        }
    }
    wp_reset_postdata();
}

// Handle form submission
function bp_wc_products_handle_submission() {
    // Check nonce for security
    check_admin_referer('bp_wc_products_create');

    // Get form data
    $title = sanitize_text_field($_POST['product-title']);
    $description = sanitize_textarea_field($_POST['product-description']);
    $price = sanitize_text_field($_POST['product-price']);
    $category = sanitize_text_field($_POST['product-category']);

    // Create new WooCommerce product
    $product = new WC_Product();
    $product->set_name($title);
    $product->set_description($description);
    $product->set_status("publish");  // can be publish,draft or any wordpress post status
    $product->set_catalog_visibility('visible'); // add the product visibility status
    $product->set_price($price);
    $product->set_regular_price($price);
    $product->set_category_ids(array($category)); // assign the product category
    $product_id = $product->save();

    // Handle product image upload and set it as the product thumbnail
    if (!function_exists('wp_handle_upload')) {
        require_once(ABSPATH . 'wp-admin/includes/file.php');
    }

    $uploadedfile = $_FILES['product-image'];
    $upload_overrides = array('test_form' => false);
    $movefile = wp_handle_upload($uploadedfile, $upload_overrides);
    $image_url = $movefile['url'];

    $upload_dir = wp_upload_dir();
    $image_data = file_get_contents($image_url);
    $filename = basename($image_url);
    if (wp_mkdir_p($upload_dir['path'])) {
        $file = $upload_dir['path'] . '/' . $filename;
    } else {
        $file = $upload_dir['basedir'] . '/' . $filename;
    }
    file_put_contents($file, $image_data);

    $wp_filetype = wp_check_filetype($filename, null);
    $attachment = array(
        'post_mime_type' => $wp_filetype['type'],
        'post_title' => sanitize_file_name($filename),
        'post_content' => '',
        'post_status' => 'inherit'
    );
    $attach_id = wp_insert_attachment($attachment, $file, $product_id);
    require_once(ABSPATH . 'wp-admin/includes/image.php');
    $attach_data = wp_generate_attachment_metadata($attach_id, $file);
    wp_update_attachment_metadata($attach_id, $attach_data);
    set_post_thumbnail($product_id, $attach_id);
    
   
    // Display success message
    echo '<p>' . __('Product created successfully!', 'buddypress-woocommerce-products') . '</p>';
}

// Load plugin text domain
function bp_wc_products_load_textdomain() {
    load_plugin_textdomain('buddypress-woocommerce-products', false, basename(dirname(__FILE__)) . '/languages'); 
}
add_action('plugins_loaded', 'bp_wc_products_load_textdomain');
