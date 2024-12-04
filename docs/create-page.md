---
description: A beautiful, reusable component for creating tokens on Zora.
---

# Create Page

The `<CreatePage />` component is a beautifully designed, ready-to-use component for creating tokens on Zora. Built with Tailwind CSS, it provides a seamless token creation experience that you can easily integrate into your applications.

## Features

- **Beautiful Design**: Modern, responsive UI built with Tailwind CSS
- **Web3-Ready**: Integrated with Zora's smart contracts and SDK
- **Wallet Connection**: Built-in wallet connection handling
- **Media Upload**: Supports image, video, and audio uploads
- **Customizable**: Easy to style and adapt to your brand
- **Open Source**: Free to use and modify

## Preview

[Preview image coming soon]

## Installation

```bash
npm install @myco/create wagmi viem @zoralabs/zora-721-contracts
```

## Required Setup

### 1. Environment Variables

Create a `.env.local` file in your project root:

```bash
NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID=your_project_id
NEXT_PUBLIC_ALCHEMY_ID=your_alchemy_id
```

### 2. Tailwind CSS

Ensure Tailwind CSS is installed and configured in your project. Add the following to your `tailwind.config.js`:

```js
module.exports = {
  content: [
    // ... your existing content
    "./node_modules/@myco/create/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### 3. Provider Setup

Wrap your application with the required providers:

```tsx
import { WagmiProvider, ZoraCreateProvider } from "@myco/create";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <WagmiProvider>
      <ZoraCreateProvider>{children}</ZoraCreateProvider>
    </WagmiProvider>
  );
}
```

## Basic Usage

The simplest way to use the `<CreatePage />` component:

```tsx
import { CreatePage } from "@myco/create";

export default function Create() {
  return <CreatePage />;
}
```

## Component API

The `<CreatePage />` component accepts the following props:

### Required Props

None - The component works out of the box with zero configuration.

### Optional Props

| Prop            | Type                        | Default        | Description                                              |
| --------------- | --------------------------- | -------------- | -------------------------------------------------------- |
| `className`     | `string`                    | `''`           | Additional CSS classes to apply to the root element      |
| `onSuccess`     | `(tokenId: string) => void` | `undefined`    | Callback function called after successful token creation |
| `defaultValues` | `CreatePageDefaultValues`   | `undefined`    | Pre-filled values for the creation form                  |
| `theme`         | `CreatePageTheme`           | `defaultTheme` | Custom theme configuration                               |

### Types

```tsx
interface CreatePageDefaultValues {
  name?: string;
  description?: string;
  symbol?: string;
  sellerFeeBasisPoints?: number;
}

interface CreatePageTheme {
  colors?: {
    primary?: string;
    secondary?: string;
    accent?: string;
    background?: string;
  };
  borderRadius?: string;
  fontFamily?: string;
}
```

## Customization

### Custom Styling

You can customize the appearance using Tailwind CSS classes:

```tsx
<CreatePage className="max-w-4xl mx-auto bg-slate-50 p-8 rounded-xl" />
```

### Custom Theme

Apply your brand colors and styling:

```tsx
<CreatePage
  theme={{
    colors: {
      primary: "#FF0000",
      secondary: "#00FF00",
      accent: "#0000FF",
      background: "#FFFFFF",
    },
    borderRadius: "1rem",
    fontFamily: "Inter",
  }}
/>
```
