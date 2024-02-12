# Payload E-Commerce Template

This is the official [Payload E-Commerce Template](https://github.com/payloadcms/payload/blob/main/templates/ecommerce). Use it to power e-commerce businesses and online stores of all sizes. This repo includes a fully-working backend, enterprise-grade admin panel, and a beautifully designed, production-ready website.

This template is right for you if you are selling:

- Physical products like clothing or merchandise
- Digital assets like ebooks or videos
- Access to content like courses or premium articles

Core features:

- [Pre-configured Payload Config](#how-it-works)
- [Authentication](#users-authentication)
- [Access Control](#access-control)
- [Shopping Cart](#shopping-cart)
- [Checkout](#checkout)
- [Paywall](#paywall)
- [Layout Builder](#layout-builder)
- [SEO](#seo)
- [Website](#website)

## Quick Start

To spin up this example locally, follow these steps:

### Clone

If you have not done so already, you need to have standalone copy of this repo on your machine. If you've already cloned this repo, skip to [Development](#development).

#### Method 1 (recommended)

  Go to Payload Cloud and [clone this template](https://payloadcms.com/new/clone/ecommerce). This will create a new repository on your GitHub account with this template's code which you can then clone to your own machine.

#### Method 2

  Use the `create-payload-app` CLI to clone this template directly to your machine:

    npx create-payload-app@latest my-project -t ecommerce

#### Method 3

  Use the `git` CLI to clone this template directly to your machine:

    git clone -n --depth=1 --filter=tree:0 https://github.com/payloadcms/payload my-project && cd my-project && git sparse-checkout set --no-cone templates/ecommerce && git checkout && rm -rf .git && git init && git add . && git mv -f templates/ecommerce/{.,}* . && git add . && git commit -m "Initial commit"

### Development

1. First [clone the repo](#clone) if you have not done so already
1. `cd my-project && cp .env.example .env` to copy the example environment variables
1. `yarn && yarn dev` to install dependencies and start the dev server
1. `open http://localhost:3000` to open the app in your browser

That's it! Changes made in `./src` will be reflected in your app. Follow the on-screen instructions to login and create your first admin user. To begin accepting payment, follow the [Stripe](#stripe) guide. Then check out [Production](#production) once you're ready to build and serve your app, and [Deployment](#deployment) when you're ready to go live.

## How it works

The Payload config is tailored specifically to the needs of most e-commerce businesses. It is pre-configured in the following ways:

### Collections

See the [Collections](https://payloadcms.com/docs/configuration/collections)  docs for details on how to extend this functionality.

- #### Users (Authentication)

  Users are auth-enabled and encompass both admins and customers based on the value of their `roles` field. Only `admin` users can access your admin panel to manage your store whereas `customer` can authenticate on your front-end to create [shopping carts](#shopping-cart) and place [orders](#orders) but have limited access to the platform. See [Access Control](#access-control) for more details.

  For additional help, see the official [Auth Example](https://github.com/payloadcms/payload/tree/main/examples/auth) or the [Authentication](https://payloadcms.com/docs/authentication/overview#authentication-overview) docs.

- #### Products

  Products are linked to Stripe via a custom select field that is dynamically populated in the sidebar of each product. This field fetches all available products in the background and displays them as options. Once a product has been selected, prices get automatically synced between Stripe and Payload through [Payload Hooks](https://payloadcms.com/docs/hooks) and [Stripe Webhooks](https://stripe.com/docs/webhooks). See [Stripe](#stripe) for more details.

  All products are layout builder enabled so you can generate unique pages for each product using layout building blocks, see [Layout Builder](#layout-builder) for more details.

  Products can also restrict access to content or digital assets behind a paywall (gated content), see [Paywall](#paywall) for more details.

- #### Orders

  Orders are created when a user successfully completes a checkout. They contain all the data about the order including the products purchased, the total price, and the user who placed the order. See [Checkout](#checkout) for more details.

- #### Pages

  All pages are layout builder enabled so you can generate unique layouts for each page using layout-building blocks, see [Layout Builder](#layout-builder) for more details.

- #### Media

  This is the uploads enabled collection used by products and pages to contain media like images, videos, downloads, and other assets.

- #### Categories

  A taxonomy used to group products together. Categories can be nested inside of one another, for example "Courses > Technology". See the official [Payload Nested Docs Plugin](https://github.com/payloadcms/plugin-nested-docs) for more details.

### Globals

See the [Globals](https://payloadcms.com/docs/configuration/globals) docs for details on how to extend this functionality.

- `Header`

  The data required by the header on your front-end like nav links.

- `Footer`

  Same as above but for the footer of your site.

## Access control

Basic role-based access control is setup to determine what users can and cannot do based on their roles, which are:

- `admin`: They can access the Payload admin panel to manage your store. They can see all data and make all operations.
- `customer`: They cannot access the Payload admin panel and can perform limited operations based on their user (see below).

This applies to each collection in the following ways:

- `users`: Only admins and the user themselves can access their profile. Anyone can create a user but only admins can delete users.
- `products`: Everyone can access products, but only admins can create, update, or delete them. Paywall-enabled products may also have content that is only accessible to only users who have purchased the product. See [Paywall](#paywall) for more details.

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

## Stripe

Payload itself handles no currency exchange. All payments are processed and billed using [Stripe](https://stripe.com). This means you must have access to a Stripe account via an API key, see [Connect Stripe](#connect-stripe) for how to get one. When you create a product in Payload that you wish to sell, it must be connected to a Stripe product by selecting one from the field in the product's sidebar, see [Products](#products) for more details. Once set, data is automatically synced between the two platforms in the following ways:

1. Stripe to Payload using [Stripe Webhooks](https://stripe.com/docs/webhooks):
   - `product.created`
   - `product.updated`
   - `price.updated`

1. Payload to Stripe using [Payload Hooks](https://payloadcms.com/docs/hooks/overview):
   - `user.create`

For more details on how to extend this functionality, see the the official [Payload Stripe Plugin](https://github.com/payloadcms/plugin-stripe).

### Connect Stripe

To integrate with Stripe, follow these steps:

1. You will first need to create a [Stripe](https://stripe.com) account if you do not already have one.
1. Retrieve your [Stripe API keys](https://dashboard.stripe.com/test/apikeys) and paste them into your `env`:
   ```bash
   STRIPE_SECRET_KEY=
   NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
   ```
1. In another terminal, listen for webhooks (optional):
   ```bash
   stripe login # follow the prompts
   yarn stripe:webhooks
   ```
1. Paste the given webhook signing secret into your `env`:
   ```bash
   STRIPE_WEBHOOKS_SIGNING_SECRET=
   ```
1. Reboot Payload to ensure that Stripe connects and the webhooks are registered.

## Checkout

A custom endpoint is opened at `POST /api/create-payment-intent` which initiates the checkout process. This endpoint totals your cart and creates a [Stripe Payment Intent](https://stripe.com/docs/payments/payment-intents). The total price is recalculated on the server to ensure accuracy and security, and once completed, passes the `client_secret` back in the response for your front-end to finalize the payment. Once the payment has succeeded, an [Order](#orders) will be created in Payload with a `stripePaymentIntentID`. Each purchased product will be recorded to the user's profile, and the user's cart will be automatically cleared.

## Paywall

Products can optionally restrict access to content or digital assets behind a paywall. This will require the product to be purchased before it's data and resources are accessible. To do this, a `purchases` field is maintained on each user to track their purchase history:

```ts
{
  name: 'purchases',
  label: 'Purchases',
  type: 'array',
  fields: [
    {
      name: 'product',
      label: 'Product',
      type: 'relationship',
      relationTo: 'products',
    },
    // other metadata like `createdOn`, etc
  ]
}
```

Then, a `paywall` field is added to the `product` with `read` access control set to check for associated purchases. Every time a user requests a product, this will only return data to those who have purchased it:

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

## Layout Builder

Create unique product and page layouts for any type fo content using a powerful layout builder. This template comes pre-configured with the following layout building blocks:

- Hero
- Content
- Media
- Call To Action
- Archive

Each block is fully designed and built into the front-end website that comes with this template. See [Website](#website) for more details.

## Production

To run Payload in production, you need to build and serve the Admin panel. To do so, follow these steps:

1. Invoke the `payload build` script by running `yarn build` or `npm run build` in your project root. This creates a `./build` directory with a production-ready admin bundle.
1. Finally run `yarn serve` or `npm run serve` to run Node in production and serve Payload from the `./build` directory.
1. When you're ready to go live, see [Deployment](#deployment) for more details.

### Deployment

Before deploying your app, you need to:

1. Switch [your Stripe account to live mode](https://stripe.com/docs/test-mode) and update your [Stripe API keys](https://dashboard.stripe.com/test/apikeys). See [Connect Stripe](#connect-stripe) for more details.
1. Ensure your app builds and serves in production. See [Production](#production) for more details.

The easiest way to deploy your project is to use [Payload Cloud](https://payloadcms.com/new/import), a one-click hosting solution to deploy production-ready instances of your Payload apps directly from your GitHub repo. You can also deploy your app manually, check out the [deployment documentation](https://payloadcms.com/docs/production/deployment) for full details.

