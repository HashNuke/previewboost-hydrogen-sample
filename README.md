# Previewboost Shopify Hydrogen sample

This guide is for using [PreviewBoost link preview images](https://previewboost.com) for your Shopify store when using any Hydrogen-based theme. These link preview images are opengraph images generated by PreviewBoost, for your Shopify store.

## Prerequisites

Your Shopify store needs to have [Previewboost](https://previewboost.com) Shopify app installed.

## Resources supported for link preview images

The PreviewBoost link preview image url is stored in a metafield for the following resources:

* Product
* Collection
* Page
* Article

## Adding support for link preview images

For fetching the link preview images for any of the above resources, the following changes are required in the routes file for each resource.

1. Update the graphql query to fetch the PreviewBoost metafield
2. Update the `meta` export in the route file to populate the meta tags for `og:image` and `og:image:secure_url`

## Updating GraphQL queries in routes

For any supported graphql resource, fetch the `og_image_url` meta field under the `app--123` namespace (notice the double dash). This is the PreviewBoost namespace for metafields on Shopify. This cannot be changed.

Below is an example for fetching products along with the metafield. You will notice that we already made the change in <app/routes/products.$handle.tsx>.

We are aliasing the metafield as `og_image_url`, so that we avoid conflict if you are already fetching other metafields in your query.

```graphql
query SampleQuery {
  products(first: 10) {
    edges {
      node {
        #
        # rest of the query
        #

        og_image_url: metafield(namespace: "app--123", key: "og_image_url") {
          value
        }

        # rest of the query
      }
    }
  }
}
```

## Updating `meta` export in the route for meta tags

Since we fetched the `og_image_url` metafield, the value should be available for us to use in the `meta` function that is exported from the route file you are working on.

Example: For a new Hydrogen theme, the default products route `meta` export looks like below:

```typescript
export const meta: MetaFunction<typeof loader> = ({data}) => {
  return [
    {title: `Hydrogen | ${data?.product.title ?? ''}`},
    {
      rel: 'canonical',
      href: `/products/${data?.product.handle}`,
    },
  ]
}
```

We are going to update it to look like this:

```typescript
export const meta: MetaFunction<typeof loader> = ({data}) => {
  const metaTags = [
    {title: `Hydrogen | ${data?.product.title ?? ''}`},
    {
      rel: 'canonical',
      href: `/products/${data?.product.handle}`,
    },
    {
      property: 'og:title',
      content: `${data?.product.title ?? ''}`,
    },
  ];

  // IMPORTANT: We are using unshift and not push.
  // unshift prepends the item to the start of the array.
  // We want these meta tags to appear on top, before any other meta tag.
  if (data?.product?.og_image_url?.value) {
    metaTags.unshift({
      property: 'og:image:secure_url',
      content: data.product.og_image_url.value,
    });
    metaTags.unshift({
      property: 'og:image',
      content: data.product.og_image_url.value,
    });
  }

  return metaTags;
};
```

Similar changes would have to be made for any other pages that require opengraph images. Check samples below.

> This guide only shows how to add the `og:image` and `og:image:secure_url`. We are assuming that you already are setting `og:title` and `og:description` in your meta tags. If not, please do so.

## References/samples

This repo contains a Hydrogen theme to which we've already made changes to support PreviewBoost link preview images. The updated files and the change summary are below.

> **IMPORTANT**: We specifically use `unshift` when adding the `meta` tags, because we want the Previewboost meta tags to appear on top, before any other meta tag.

### <app/routes/collections.$handle.tsx>

* Added `og_image_url` metafield to fetch in the collection query.
* Updated the `meta` export in the file, to set the `og:image` and `og:image:secure_url` if available.

### <app/routes/products.$handle.tsx>

* Added `og_image_url` metafield to fetch in the the Product fragment.
* Updated the `meta` export in the file, to set the `og:image` and `og:image:secure_url` if available.

### <app/routes/pages.$handle.tsx>

* Updated the page query to fetch the `og_image_url` metafield.
* Updated the `meta` export in the file, to set the `og:image` and `og:image:secure_url` if available.

###  <app/routes/blogs.$blogHandle.$articleHandle.tsx>

* Updated the article query to fetch the `og_image_url` metafield.
* Updated the `meta` export in the file, to set the `og:image` and `og:image:secure_url` if available.

## How to verify if the meta tags are working

When your changes are complete, and if Previewboost has generated images already, then viewing your HTML source to check if the `og:image` tags are visible should suffice. You can also use [Facebook Sharing debugger](https://developers.facebook.com/tools/debug/) to check.

## Need help?

If you are building a Hydrogen theme or headless storefront for your Shopify store and need help with using PreviewBoost, reach out to us at <mailto:hello@previewboost.com>
