---
description: A beautiful, reusable component for creating tokens on Zora.
---

# Create Page

The `<CreatePage />` component is a beautifully designed, ready-to-use component for creating tokens on Zora. Built with Tailwind CSS, it provides a seamless token creation experience that you can easily integrate into your applications.

## Features

- **Beautiful Design**: Modern, responsive UI built with Tailwind CSS
- **Onchain-Ready**: Integrated with Zora's smart contracts and SDK
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

Create the following files in your project:

#### `providers/wagmi-provider.tsx`

```tsx
"use client";

import { createConfig, WagmiConfig, http } from "wagmi";
import { base } from "wagmi/chains";

const config = createConfig({
  chains: [base],
  transports: {
    [base.id]: http(),
  },
});

export function WagmiProvider({ children }: { children: React.ReactNode }) {
  return <WagmiConfig config={config}>{children}</WagmiConfig>;
}
```

#### `providers/zora-create-provider.tsx`

```tsx
"use client";

import { createPublicClient, http } from "viem";
import { base } from "viem/chains";
import { createContext, useContext, useState } from "react";
import { zoraCreator721Factory } from "@zoralabs/zora-721-contracts";

interface ZoraCreateContextType {
  createToken: (params: CreateTokenParams) => Promise<string>;
  isLoading: boolean;
  error: Error | null;
}

interface CreateTokenParams {
  name: string;
  symbol: string;
  description?: string;
  sellerFeeBasisPoints?: number;
  mediaUrl?: string;
}

const ZoraCreateContext = createContext<ZoraCreateContextType>({
  createToken: async () => "",
  isLoading: false,
  error: null,
});

export function ZoraCreateProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const publicClient = createPublicClient({
    chain: base,
    transport: http(),
  });

  const createToken = async (params: CreateTokenParams) => {
    setIsLoading(true);
    setError(null);
    try {
      const { hash } = await publicClient.writeContract({
        ...zoraCreator721Factory,
        functionName: "createToken",
        args: [
          params.name,
          params.symbol,
          params.description || "",
          params.sellerFeeBasisPoints || 0,
          params.mediaUrl || "",
        ],
      });
      const receipt = await publicClient.waitForTransactionReceipt({ hash });
      return receipt.transactionHash;
    } catch (err) {
      setError(err as Error);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <ZoraCreateContext.Provider value={{ createToken, isLoading, error }}>
      {children}
    </ZoraCreateContext.Provider>
  );
}

export const useZoraCreate = () => useContext(ZoraCreateContext);
```

Then wrap your application with these providers:

#### `app/layout.tsx`

```tsx
import { WagmiProvider } from "@/providers/wagmi-provider";
import { ZoraCreateProvider } from "@/providers/zora-create-provider";

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

These providers set up:

- Wallet connection handling with Wagmi
- Zora contract interactions
- Loading and error states
- Type-safe context for token creation

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

## Examples

### Basic Implementation

```tsx
import { CreatePage } from "@myco/create";

export default function CreateToken() {
  const handleSuccess = (tokenId: string) => {
    console.log(`Token ${tokenId} created successfully!`);
  };

  return (
    <CreatePage onSuccess={handleSuccess} className="max-w-4xl mx-auto p-8" />
  );
}
```

### Pre-filled Values

```tsx
import { CreatePage } from "@myco/create";

export default function CreateCollectionToken() {
  return (
    <CreatePage
      defaultValues={{
        name: "My Collection",
        symbol: "MYCOL",
        sellerFeeBasisPoints: 500, // 5%
      }}
    />
  );
}
```

### Custom Success Handler

```tsx
import { CreatePage } from "@myco/create";
import { useRouter } from "next/navigation";

export default function CreateWithRedirect() {
  const router = useRouter();

  const handleSuccess = (tokenId: string) => {
    router.push(`/collection/${tokenId}`);
  };

  return <CreatePage onSuccess={handleSuccess} />;
}
```

## Troubleshooting

### Common Issues

#### Wallet Connection Issues

If you're experiencing wallet connection problems:

1. Ensure you've set up the `NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID` environment variable
2. Check that the `WagmiProvider` is properly configured
3. Verify that your wallet is connected to the correct network

#### Transaction Failures

If token creation transactions are failing:

1. Ensure the user has enough ETH for gas fees
2. Check that all required fields are properly filled
3. Verify network connectivity

#### Styling Issues

If the component styling doesn't match your design:

1. Verify that Tailwind CSS is properly configured
2. Check that the content array in `tailwind.config.js` includes the component path
3. Ensure your theme configuration is properly structured

### Getting Help

- Check the [GitHub repository](https://github.com/sweetmantech/myco-create) for known issues
- Join our [Discord community](https://discord.gg/myco) for support
- Submit bug reports through GitHub Issues

## Contributing

We welcome contributions! Please see our [Contributing Guide](https://github.com/sweetmantech/myco-create/CONTRIBUTING.md) for details on how to get started.
