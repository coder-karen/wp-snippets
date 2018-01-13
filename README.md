# Wordpress Code Snippets

Here is a collection of code snippets that I use and have found helpful.

* [Custom Page Titles](#custom-page-titles)
* [Custom Page Titles with Yoast](#custom-page-titles-with-yoast)
* [Customising the Custom Post Type Query](#customising-the-custom-post-type-query)
* [Enabling Shortcodes within the Custom HTML Widget](#enabling-shortcodes-within-the-custom-html-widget)
* [Send Post Comments to Custom Email Addresses](#send-post-comments-to-custom-email-addresses)

### Custom Page Titles
Assuming you have added theme support for title-tag.

```php
function custom_theme_titles( $titleparts ) {
    if ( is_front_page() ) {
		$pagetitle = $titleparts['title'];
        $titleparts['title'] = $pagetitle;
        unset( $titleparts['site'] );
    }

    elseif ( is_404() ) {
    	$pagetitle = "404";
     	$sep = " | ";
        $name = get_bloginfo( 'name', 'display' );
        $sep2 = " - ";
        $desc = get_bloginfo( 'description', 'display' );
        $titleparts['title'] = $pagetitle.$sep.$name.$sep2.$desc;
        unset( $titleparts['site'] );
    }

    elseif (is_category() ) {
		$category = get_the_category();
		$catName = $category[0]->cat_name;		
		$pagetitle = "Blog - ".$catName." archives";		
    	$sep = " | ";
        $name = get_bloginfo( 'name', 'display' );
        $sep2 = " - ";
        $desc = get_bloginfo( 'description', 'display' );
        $titleparts['title'] = $pagetitle.$sep.$name.$sep2.$desc;
        unset( $titleparts['site'] );
    }

    else {
  		$pagetitle = $titleparts['title'];
        $sep = " | ";
        $name = get_bloginfo( 'name', 'display' );
        $sep2 = " - ";
        $desc = get_bloginfo( 'description', 'display' );
        $titleparts['title'] = $pagetitle.$sep.$name.$sep2.$desc;
        unset( $titleparts['site'] );
    }
    return $titleparts;
}

add_filter( 'document_title_parts', 'custom_theme_titles', PHP_INT_MAX );
```


### Custom Page Titles with Yoast
For creating custom page titles if you have Yoast SEO installed and activated.

```php
function custom_theme_titles_yoast( $title ) {
    $sep = '|';
    $name = get_bloginfo( 'name' );
	$desc = get_bloginfo( 'description', 'display' );

    if( is_front_page() ) {
		$pagetitle = $title;
		return "{$pagetitle}";
    }

     elseif( is_404() ) {
  		$pagetitle = "404";
		return "{$pagetitle} {$sep} {$name} - {$desc}";
    }

    else {
		$pagetitle = $title;   
		return "{$pagetitle} {$sep} {$name} - {$desc}";
    }
}

add_filter( 'wpseo_title', 'custom_theme_titles_yoast', PHP_INT_MAX );
```


### Customising the Custom Post Type Query
Working with pre_get_posts to modify custom post type (events) output on a page (only displaying posts that have a checkbox checked (using the advanced custom fields plugin) and only showing a maximum of 3 posts per page).

```php
function ka_event_query( $query ) {
	if( $query->is_main_query() && !$query->is_feed() && !is_admin() && $query->is_post_type_archive( 'events' ) ) {
		//Filtering out posts that have checkbox checked (therefore marked as true)
		$meta_query = array(
			array(
				'key' => 'event_current', //key as created in advanced custom field
				'value' => true,
				'compare' => 'BOOLEAN'
			)
		);
		$query->set( 'meta_query', $meta_query );
		 //Display only 3 posts
		$query->set( 'posts_per_page', '3' );
	}
}

add_action( 'pre_get_posts', 'ka_event_query' );

```


### Enabling Shortcodes within the Custom HTML Widget
Enabling shortcodes within the Text Widget does not transfer over to the newer Custom HTML Widget. This allows shortcodes in the Custom HTML Widget.

```php
add_filter('widget_custom_html_content','do_shortcode');

```


### Send Post Comments to Custom Email Addresses
Here is a solution to make sure all post comments go to the post author (allowing for custom post-types as well) and a second specified email address. Alternatively just leave the specified email address as the only option.

```php
function ka_comment_moderation_recipients( $emails, $comment_id ) {
    $comment = get_comment( $comment_id );

    if(empty($comment)) {
        return $emails;
    }

    $post = get_post( $comment->comment_post_ID );

    // Refine if you only want comments from a custom post type (or remove this section)
    if($post->post_type != "custom_post_type") {
        return $emails;
    }

    $user = get_user_by( 'id', $post->post_author );
 
    // Return the post author email if the author can modify, as well as a custom email
    if ( user_can( $user->ID, 'edit_published_posts' ) && ! empty( $user->user_email ) ) {
        $emails = array( $user->user_email, 'info@example.com');
    }

    //Else return just the custom email
    else {
        $emails = array('info@example.com');
    }
 
    return $emails;
}

add_filter( 'comment_moderation_recipients', 'ka_comment_moderation_recipients', 11, 2 );
add_filter( 'comment_notification_recipients', 'ka_comment_moderation_recipients', 11, 2 );

```