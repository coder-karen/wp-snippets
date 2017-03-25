# Wordpress Code Snippets

Here is a collection of code snippets that I use and have found helpful.

* [Custom Page Titles](#custom-page-titles)
* [Custom Page Titles with Yoast](#custom-page-titles-with-yoast)
* [Customising the Custom Post Type Query](#customising-the-custom-post-type-query)

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
Working with pre_get_posts to modify custom post type (events) output on a page (only displaying posts that have a checkbox checked (using the advanced custom fields plugin) and only showing a maximum of 3 posts per page)

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