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
npm install wagmi viem @zoralabs/zora-721-contracts
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
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### 3. Create Required Files

#### Providers

First, create the necessary providers:

##### `providers/wagmi-provider.tsx`

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

##### `providers/zora-create-provider.tsx`

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
          0,
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

#### Hooks

Create the following hooks:

##### `hooks/use-media-upload.tsx`

```tsx
"use client";

import { useState } from "react";

interface UseMediaUploadResult {
  uploadMedia: (file: File) => Promise<string>;
  isUploading: boolean;
  error: Error | null;
}

export function useMediaUpload(): UseMediaUploadResult {
  const [isUploading, setIsUploading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const uploadMedia = async (file: File): Promise<string> => {
    setIsUploading(true);
    setError(null);
    try {
      // Implement your media upload logic here
      // This could be to IPFS, Arweave, or your preferred storage
      const url = await uploadToStorage(file);
      return url;
    } catch (err) {
      setError(err as Error);
      throw err;
    } finally {
      setIsUploading(false);
    }
  };

  return { uploadMedia, isUploading, error };
}

// Example implementation - replace with your preferred storage solution
async function uploadToStorage(file: File): Promise<string> {
  // Implement your storage upload logic here
  return "https://example.com/media/1234";
}
```

#### Components

First, create some utility components:

##### `components/ui/spinner.tsx`

```tsx
export default function Spinner() {
  return (
    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-black" />
  );
}
```

##### `lib/utils.ts`

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

export function getIpfsLink(uri?: string) {
  if (!uri) return "";
  if (uri.startsWith("ipfs://")) {
    return uri.replace("ipfs://", "https://ipfs.io/ipfs/");
  }
  return uri;
}
```

##### `components/media-upload/no-file-selected.tsx`

```tsx
interface NoFileSelectedProps {
  onClick: () => void;
}

export default function NoFileSelected({ onClick }: NoFileSelectedProps) {
  return (
    <div
      onClick={onClick}
      className="absolute inset-0 flex flex-col items-center justify-center cursor-pointer"
    >
      <svg
        className="w-8 h-8 mb-4 text-gray-500"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          strokeLinecap="round"
          strokeLinejoin="round"
          strokeWidth={2}
          d="M12 6v6m0 0v6m0-6h6m-6 0H6"
        />
      </svg>
      <span className="text-gray-500">Click to upload media</span>
    </div>
  );
}
```

##### `components/media-upload/audio-player.tsx`

```tsx
interface AudioPlayerProps {
  onClick: () => void;
}

export default function AudioPlayer({ onClick }: AudioPlayerProps) {
  return (
    <div
      onClick={onClick}
      className="absolute inset-0 flex items-center justify-center cursor-pointer"
    >
      <svg
        className="w-16 h-16 text-gray-500"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          strokeLinecap="round"
          strokeLinejoin="round"
          strokeWidth={2}
          d="M9 19V5l12 7-12 7z"
        />
      </svg>
    </div>
  );
}
```

##### `components/media-upload/media-upload.tsx`

```tsx
"use client";

import { useZoraCreate } from "@/providers/zora-create-provider";
import { useMediaUpload } from "@/hooks/use-media-upload";
import { cn, getIpfsLink } from "@/lib/utils";
import Spinner from "@/components/ui/spinner";
import { useRef } from "react";
import NoFileSelected from "./no-file-selected";
import AudioPlayer from "./audio-player";
import Image from "next/image";

export default function MediaUpload() {
  const { imageUri, animationUri, mimeType } = useZoraCreate();
  const {
    uploadMedia: fileUpload,
    isUploading: loading,
    error,
    blurImageUrl,
  } = useMediaUpload();
  const fileInputRef = useRef<HTMLInputElement>(null);

  const handleImageClick = () => {
    fileInputRef.current?.click();
  };

  const renderMedia = () => {
    if (loading) {
      return (
        <div className="absolute inset-0 flex items-center justify-center left-0 top-0">
          <Spinner />
        </div>
      );
    }

    if (mimeType?.includes("audio")) {
      return <AudioPlayer onClick={handleImageClick} />;
    }

    if (mimeType?.includes("video")) {
      return (
        <video controls className="w-full rounded-md">
          <source src={getIpfsLink(animationUri)} type={mimeType} />
          Your browser does not support the video element.
        </video>
      );
    }

    if (imageUri) {
      return (
        <div className="relative w-[296px] h-[296px]">
          <Image
            src={blurImageUrl || getIpfsLink(imageUri)}
            className="w-full h-auto rounded-md cursor-pointer object-contain absolute"
            alt="Image Preview"
            onClick={handleImageClick}
            blurDataURL={blurImageUrl}
            fill
          />
        </div>
      );
    }

    return <NoFileSelected onClick={handleImageClick} />;
  };

  return (
    <div className="grid w-full max-w-3xl items-center gap-4">
      <div
        className={cn(
          "w-full relative rounded-md min-h-[300px] min-w-[300px]",
          !imageUri && !animationUri && "aspect-square",
          (loading || (!imageUri && !animationUri)) &&
            "border-dashed border-2 border-black"
        )}
      >
        <input
          ref={fileInputRef}
          id="media"
          type="file"
          className="hidden"
          onChange={fileUpload}
          accept="image/*,video/*,audio/*"
        />
        {renderMedia()}
      </div>
      {error && <p className="text-red-500 text-sm mt-2">{error.message}</p>}
    </div>
  );
}
```

Now update the `CreatePage` component to use `MediaUpload`:

```tsx
"use client";

import { useZoraCreate } from "@/providers/zora-create-provider";
import { useMediaUpload } from "@/hooks/use-media-upload";
import { useState } from "react";

interface CreatePageProps {
  className?: string;
  onSuccess?: (tokenId: string) => void;
  defaultValues?: CreatePageDefaultValues;
  theme?: CreatePageTheme;
}

interface CreatePageDefaultValues {
  name?: string;
  description?: string;
  symbol?: string;
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

export function CreatePage({
  className = "",
  onSuccess,
  defaultValues,
  theme,
}: CreatePageProps) {
  const {
    createToken,
    isLoading: isCreating,
    error: createError,
  } = useZoraCreate();
  const { uploadMedia, isUploading, error: uploadError } = useMediaUpload();
  const [mediaUrl, setMediaUrl] = useState("");
  const [formData, setFormData] = useState({
    name: defaultValues?.name || "",
    symbol: defaultValues?.symbol || "",
    description: defaultValues?.description || "",
  });

  const handleMediaChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    try {
      const url = await uploadMedia(file);
      setMediaUrl(url);
    } catch (err) {
      console.error("Failed to upload media:", err);
    }
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      const tokenId = await createToken({
        ...formData,
        mediaUrl,
        sellerFeeBasisPoints: 0,
      });
      onSuccess?.(tokenId);
    } catch (err) {
      console.error("Failed to create token:", err);
    }
  };

  return (
    <div className={className}>
      <form onSubmit={handleSubmit} className="space-y-6">
        <div>
          <label className="block text-sm font-medium text-gray-700">
            Media
            <MediaUpload />
          </label>
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700">
            Name
            <input
              type="text"
              value={formData.name}
              onChange={(e) =>
                setFormData((prev) => ({ ...prev, name: e.target.value }))
              }
              className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
              required
            />
          </label>
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700">
            Symbol
            <input
              type="text"
              value={formData.symbol}
              onChange={(e) =>
                setFormData((prev) => ({ ...prev, symbol: e.target.value }))
              }
              className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
              required
            />
          </label>
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700">
            Description
            <textarea
              value={formData.description}
              onChange={(e) =>
                setFormData((prev) => ({
                  ...prev,
                  description: e.target.value,
                }))
              }
              className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
            />
          </label>
        </div>

        <button
          type="submit"
          disabled={isCreating || isUploading}
          className="w-full px-4 py-2 text-white bg-blue-500 rounded-md hover:bg-blue-600 disabled:opacity-50"
        >
          {isUploading
            ? "Uploading..."
            : isCreating
            ? "Creating..."
            : "Create Token"}
        </button>

        {(createError || uploadError) && (
          <p className="text-red-500">
            {createError?.message || uploadError?.message}
          </p>
        )}
      </form>
    </div>
  );
}
```

Finally, wrap your application with the providers:

##### `app/layout.tsx`

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

These providers and components set up:

- Wallet connection handling with Wagmi
- Zora contract interactions
- Media upload functionality
- Loading and error states
- Type-safe context for token creation
- Beautiful UI with Tailwind CSS

## Basic Usage

```tsx
import { CreatePage } from "@/components/create-page";

export default function Create() {
  const handleSuccess = (tokenId: string) => {
    console.log(`Token ${tokenId} created successfully!`);
  };

  return (
    <CreatePage onSuccess={handleSuccess} className="max-w-4xl mx-auto p-8" />
  );
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
import { CreatePage } from "@/components/create-page";

export default function CreateCollectionToken() {
  return (
    <CreatePage
      defaultValues={{
        name: "My Collection",
        symbol: "MYCOL",
        description: "A beautiful collection of digital art",
      }}
    />
  );
}
```

### Custom Success Handler

```tsx
import { CreatePage } from "@/components/create-page";
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
