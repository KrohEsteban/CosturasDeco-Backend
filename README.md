# Payload E-Commerce Template

A template for [Payload CMS](https://github.com/payloadcms/payload) to power e-commerce businesses. There is a complete front-end website for this template which can be found [here](https://github.com/payloadcms/template-ecommerce-nextjs).

Core features:

- [Pre-configured Payload Config](#how-it-works)
- [Authentication](#users)
- [Access Control](#access-control)
- [Shopping Cart](#shopping-cart)
- [Checkout](#checkout)
- [Paywall](#paywall)
- [Layout Builder](#layout-builder)
- [SEO](#seo)

## How it works

The Payload config is tailored specifically to the needs of an e-commerce business. It is pre-configured in the following ways:

### Collections

- #### Users

  Users are auth-enabled and encompass both admins and customers based on the value of their `roles` field. Only `admins` can access your admin panel to manage your store, whereas `customers` can authenticate on your front-end to create [shopping carts](#shopping-cart) and place [orders](#orders)—but have limited access to the platform, see [access control](#access-control) for more details.

- #### Products

  Each product is linked to Stripe via a select field that is dynamically populated in the sidebar. This field fetches all available products in the background and displays them as options. Once a product has been selected, prices get automatically synced between Stripe and Payload. All products are layout-builder enabled so you can generate unique pages for each product using layout-building blocks, see [Layout Builder](#layout-builder) for more details. Products can also gate their content or digital assets behind a paywall, see [Paywall](#paywall) for more details.

- #### Orders

  When an order is placed in Stripe, a webhook is fired that Payload listens for. This webhook creates a new order in Payload with the same data as the invoice. See [Stripe Integration](#stripe-integration) for more details.

- #### Pages

  All pages are layout-builder enabled so you can generate unique layouts for each page using layout-building blocks, see [Layout Builder](#layout-builder) for more details.

- #### Media

  This is the uploads-enabled collection used by products and page to contain media, etc.

- #### Categories

  A taxonomy used to group products together. Categories can be nested inside of one another, for example "Shirts > Red". See the official [Payload Nested Docs Plugin](https://github.com/payloadcms/plugin-nested-docs) for more details.

### Globals

- `Header`

  The data required by the header on your front-end, i.e. nav links, etc.

- `Footer`

  Same as above but for the footer of your site.

## Access control

Basic role-based access control is setup to determine what users can and cannot do based on their roles, which are:

- `admin`: They can access the Payload admin panel to manage your store. They can see all data and make all operations.
- `customer`: They cannot access the Payload admin panel and have a limited access to operations based on their user (see below).

This applies to each collection in the following ways:

- `users`: Only admins and the user themselves can access their profile. Anyone can create a user but only admins can delete users.
- `orders`: Only admins and the user who placed the order can access it. Once placed, orders cannot be edited or deleted.
- `products`: Everyone can access products, but only admins can create, update, or delete them. Paywall-enabled products may also have content that is only accessible by users who have purchased the product. See [Paywall](#paywall) for more details.

For more details on how to extend this functionality, see the [Payload Access Control](https://payloadcms.com/docs/access-control/overview#access-control) docs.

## Shopping cart

Logged-in users can have their shopping carts saved to their profiles as they shop. This way they can continue shopping at a later date or on another device. When not logged in, the cart can be saved to local storage and synced to Payload on the next login. This works by maintaining a `cart` field on the `user`:

```ts
{
  name: 'cart',
  label: 'Shopping Cart',
  type: 'object',
  fields: [
    {
      name: 'items',
      label: 'Items',
      type: 'array',
      fields: [
        // product, quantity, etc
      ]
    },
    // other metadata like `createdOn`, etc
  ]
}
```

A complete front-end solution for this can be found [here](https://github.com/payloadcms/template-ecommerce-nextjs).

## Stripe integration

Payload itself handles no currency exchange. All payments are processed and billed using [Stripe](https://stripe.com). This means you must have access to a Stripe account via an API key, see [Connect Stripe](#connect-stripe) for how to get one. When you create a product in Payload that wish to sell, it must be connected to a Stripe product by selecting one from the field in the products sidebar. This field fetches all available products in the background and displays them as options, see [Products](#products) for more details. Once set, data is automatically synced between the two platforms in the following ways:

1. Stripe to Payload using [Stripe Webhooks](https://stripe.com/docs/webhooks):

   - `invoice.created`
   - `invoice.updated`
   - `product.created`
   - `product.updated`
   - `price.updated`

1. Payload to Stripe using [Payload Hooks](https://payloadcms.com/docs/hooks/overview):
   - `user.create`

For more details on how to extend this functionality, see the the official [Payload Stripe Plugin](https://github.com/payloadcms/plugin-stripe).

## Checkout

A custom endpoint is opened at `/api/checkout` which initiates the checkout process. This endpoint creates a [`PaymentIntent`](https://stripe.com/docs/payments/payment-intents) with the items in the cart using the Stripe's [Invoices API](https://stripe.com/docs/api/invoices). First, an invoice is drafted, then each item in your cart is appended as a line-item to the invoice. The total price is recalculated on the server to ensure accuracy and security, and once completed, passes the `client_secret` back in the response for your front-end to finalize the payment.

## Paywall

Products can optionally gate content or digital assets behind a paywall. This will require the product to be purchased before it's resources are accessible. To do this, we add a `paywall` field to the `product` collection with `read` access control to check for associated purchases on each request. A `purchases` field is maintained on each user to determine their access which can be manually adjusted as needed.

```ts
{
  name: 'paywall',
  label: 'Paywall',
  type: 'blocks',
  access: {
    read: checkUserPurchases,
  },
  fields: [
    // assets
  ]
}
```

## Layout builder

Products and pages can be built using a powerful layout builder. This allows you to create unique layouts for each product or page. This boilerplate comes pre-configured with the following layout building blocks:

- Hero
- Content
- Media
- Call To Action
- Archive

A complete front-end solution for this can be found [here](https://github.com/payloadcms/template-ecommerce-nextjs).

## SEO

This boilerplate comes pre-configured with the official [Payload SEO Plugin](http://payloadcms.com/) for complete SEO control. A front-end solution for this can be found [here](https://github.com/payloadcms/template-ecommerce-nextjs).

## Development

To spin up the boilerplate, follow these steps:

1.  First clone the repo
1.  Then, `cd YOUR_PROJECT_REPO && yarn && yarn dev`
1.  Now open `http://localhost:8000/admin` in your browser
1.  Create your first admin user using the form on the page

That's it! Changes made in `./src` will be reflected in your app—but your database is blank and your app is not yet connected to Stripe, more details on that [here](#stripe). You can optionally seed the database with a few products and pages, more details on that [here](#seed).

### Connect Stripe

To integrate with Stripe, follow these steps:

1. You will first need to create a [Stripe](https://stripe.com/) account if you do not already have one.
1. Retrieve your Stripe Secret Key from the Stripe admin panel and paste it into your `env`:
   ```bash
   STRIPE_SECRET_KEY=
   ```
1. In another terminal, listen for webhooks:
   ```bash
   stripe login # follow the prompts
   stripe listen --forward-to localhost:8000/stripe/webhooks
   ```
1. Paste the given webhook signing secret into your `env`:
   ```bash
   STRIPE_WEBHOOKS_ENDPOINT_SECRET=
   ```
1. Reboot Payload to ensure that Stripe connects and the webhooks are registered.

See the official [Payload Stripe Plugin](https://github.com/payloadcms/plugin-stripe) for more details.

### Seed

To seed the database with a few products and pages you can run `yarn seed`.

> NOTICE: seeding the database is destructive because it drops your current database to populate a fresh one from the seed template. Only run this command if you are starting a new project or can afford to lose your current data.
