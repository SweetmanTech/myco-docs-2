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
npm install wagmi viem @zoralabs/protocol-sdk @zoralabs/protocol-deployments clsx tailwind-merge @tanstack/react-query lucide-react
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

## Providers

First, create the necessary providers:

### WagmiProvider

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode } from "react";
import { coinbaseWallet } from "wagmi/connectors";
import { createConfig, http, WagmiProvider as WProvider } from "wagmi";
import { CHAIN, CHAIN_ID } from "@/lib/consts";

const queryClient = new QueryClient();

const config = createConfig({
  chains: [CHAIN],
  connectors: [
    coinbaseWallet({
      appName: "myco.wtf",
      preference: "smartWalletOnly",
    }),
  ],
  transports: {
    [CHAIN_ID]: http(),
  },
});

const WagmiProvider = ({ children }: { children: ReactNode }) => (
  <WProvider config={config}>
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  </WProvider>
);

export { WagmiProvider };
```

### ZoraCreateProvider

```tsx
"use client";

import useZoraCreate from "@/hooks/useZoraCreate";
import React, { createContext, useContext, useMemo } from "react";

const ZoraCreateContext =
  createContext<ReturnType<typeof useZoraCreate>>(undefined);

const ZoraCreateProvider = ({ children }: any) => {
  const zoraCreate = useZoraCreate();

  const value = useMemo(() => ({ ...zoraCreate }), [zoraCreate]);

  return (
    <ZoraCreateContext.Provider value={value}>
      {children}
    </ZoraCreateContext.Provider>
  );
};

const useZoraCreateProvider = () => {
  const context = useContext(ZoraCreateContext);
  if (!context) {
    throw new Error(
      "useZoraCreateProvider must be used within a ZoraCreateProvider"
    );
  }
  return context;
};

export { ZoraCreateProvider, useZoraCreateProvider };
```

## API

### api/ipfs/route

```tsx
import saveFile from "@/lib/ipfs/saveFile";
import { NextRequest } from "next/server";

export const runtime = "edge";

export async function POST(request: NextRequest) {
  const data = await request.formData();
  const file: File | null = data.get("file") as unknown as File;
  data.append("file", file);
  data.append("pinataMetadata", JSON.stringify({ name: "File to upload" }));
  const cid = await saveFile(data);
  return Response.json({ cid }, { status: 200 });
}
```

### api/ipfs/generateJWT/route

```tsx
import generatePinataJWT from "@/lib/ipfs/generatePinataJWT";

export async function GET() {
  try {
    const data = await generatePinataJWT();
    return Response.json(data);
  } catch (e) {
    console.log(e);
    const message = e?.message ?? "failed to generate JWT";
    return Response.json({ message }, { status: 500 });
  }
}

export const dynamic = "force-dynamic";
export const fetchCache = "force-no-store";
export const revalidate = 0;
```

## Hooks

Create the following hooks:

### useZoraCreate

```tsx
"use client";

import { useState } from "react";
import { useAccount, useSwitchChain, useWriteContract } from "wagmi";
import { CHAIN_ID } from "@/lib/consts";
import { useParams } from "next/navigation";
import { Address } from "viem";
import useZoraCreateParameters from "./useZoraCreateParameters";

export default function useZoraCreate() {
  const { address } = useAccount();
  const { writeContractAsync } = useWriteContract();
  const { switchChainAsync } = useSwitchChain();
  const [creating, setCreating] = useState<boolean>(false);
  const params = useParams();
  const chainId = Number(params.chainId) || CHAIN_ID;
  const collection = params.collection as Address | undefined;
  const { fetchParameters, createMetadata } = useZoraCreateParameters(
    chainId,
    collection
  );

  const create = async () => {
    setCreating(true);
    try {
      if (!address) {
        throw new Error("No wallet connected");
      }
      await switchChainAsync({ chainId });
      const parameters = await fetchParameters();

      if (!parameters) {
        throw new Error("Parameters not ready");
      }

      await writeContractAsync({
        ...parameters,
      });
    } catch (err) {
      setCreating(false);
      console.error(err);
    }
  };

  return { create, creating, ...createMetadata };
}
```

### useZoraCreateParameters

```tsx
import { createCreatorClient } from "@zoralabs/protocol-sdk";
import { Address } from "viem";
import { CHAIN_ID, REFERRAL_RECIPIENT } from "@/lib/consts";
import { useAccount, usePublicClient } from "wagmi";
import getSalesConfig from "@/lib/zora/getSalesConfig";
import useCreateMetadata from "@/hooks/useCreateMetadata";

const useZoraCreateParameters = (
  chainId: number = CHAIN_ID,
  collection: Address
) => {
  const publicClient = usePublicClient();
  const { address } = useAccount();
  const createMetadata = useCreateMetadata();

  const fetchParameters = async () => {
    if (!publicClient) return;
    const creatorClient = createCreatorClient({ chainId, publicClient });
    const { uri: cc0MusicIpfsHash } = await createMetadata.getUri();
    const salesConfig = getSalesConfig(
      createMetadata.isTimedSale
        ? "ZoraTimedSaleStrategy"
        : "ZoraFixedPriceSaleStrategy"
    );

    let newParameters;
    if (collection) {
      const { parameters: existingParameters } =
        await creatorClient.create1155OnExistingContract({
          contractAddress: collection,
          token: {
            tokenMetadataURI: cc0MusicIpfsHash,
            createReferral: REFERRAL_RECIPIENT,
            salesConfig,
          },
          account: address,
        });
      newParameters = existingParameters;
    } else {
      const { parameters: newContractParameters } =
        await creatorClient.create1155({
          contract: {
            name: createMetadata.name,
            uri: cc0MusicIpfsHash,
          },
          token: {
            tokenMetadataURI: cc0MusicIpfsHash,
            createReferral: REFERRAL_RECIPIENT,
            salesConfig,
          },
          account: address,
        });
      newParameters = {
        ...newContractParameters,
        functionName: "createContract",
      };
    }

    return newParameters;
  };

  return { createMetadata, fetchParameters };
};

export default useZoraCreateParameters;
```

### useCreateMetadata

```tsx
import { uploadJson } from "@/lib/ipfs/uploadJson";
import { useState } from "react";

const useCreateMetadata = () => {
  const [name, setName] = useState<string>("");
  const [isTimedSale, setIsTimedSale] = useState<boolean>(true);
  const [imageUri, setImageUri] = useState<string>("");
  const [mimeType, setMimeType] = useState<string>("");
  const [animationUri, setAnimationUri] = useState<string>("");

  const getUri = async () =>
    await uploadJson({
      name,
      description: "",
      image: imageUri,
      animation_url: animationUri,
      content: {
        mime: mimeType,
        uri: animationUri,
      },
    });

  return {
    animationUri,
    setAnimationUri,
    getUri,
    imageUri,
    setImageUri,
    mimeType,
    setMimeType,
    name,
    isTimedSale,
    setName,
    setIsTimedSale,
  };
};

export default useCreateMetadata;
```

### useFileUpload

```tsx
import { MAX_FILE_SIZE, ONE_MB } from "@/lib/consts";
import { uploadFile } from "@/lib/ipfs/uploadFile";
import { useZoraCreateProvider } from "@/providers/ZoraCreateProvider";
import { useState } from "react";

const useFileUpload = () => {
  const { setName, setImageUri, setAnimationUri, setMimeType, animationUri } =
    useZoraCreateProvider();
  const [blurImageUrl, setBlurImageUrl] = useState<string>("");
  const [error, setError] = useState<string>("");
  const [loading, setLoading] = useState<boolean>(false);

  const fileUpload = async (event) => {
    setError("");
    setLoading(true);

    try {
      const file = event.target.files[0];
      if (!file) {
        throw new Error();
      }
      if (file.size > MAX_FILE_SIZE) {
        throw new Error(
          `File size exceeds the maximum limit of ${MAX_FILE_SIZE / ONE_MB}MB.`
        );
      }

      const mimeType = file.type;
      const isImage = mimeType.includes("image");

      if (isImage) {
        const fileNameWithoutExtension = file.name.replace(/\.[^/.]+$/, "");
        setName(fileNameWithoutExtension);
      }

      const { uri } = await uploadFile(file);
      if (isImage) {
        setImageUri(uri);
        setBlurImageUrl(URL.createObjectURL(file));
        if (!animationUri) {
          setMimeType(mimeType);
        }
      } else {
        setAnimationUri(uri);
        setMimeType(mimeType);
      }
    } catch (err) {
      console.error(err);
      setError(err.message ?? "Failed to upload the file. Please try again.");
    }
    setLoading(false);
  };

  return { fileUpload, loading, error, blurImageUrl };
};

export default useFileUpload;
```

### useConnectWallet

```tsx
import { useConnect } from "wagmi";

const useConnectWallet = () => {
  const { connectors, connectAsync } = useConnect();
  const connector = connectors[0];

  const connectWallet = () => connectAsync({ connector });

  return {
    connectWallet,
  };
};

export default useConnectWallet;
```

## Components

First, create some utility components:

### ui/spinner

```tsx
export default function Spinner() {
  return (
    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-black" />
  );
}
```

### ui/button

- ShadCN Button https://ui.shadcn.com/docs/components/button

### ui/slider

- ShadCN Slider https://ui.shadcn.com/docs/components/slider

### ui/input

- ShadCN Input https://ui.shadcn.com/docs/components/input

### ui/label

- ShadCN Label https://ui.shadcn.com/docs/components/label

### MediaUpload/UploadIcon

```tsx
const UploadIcon = (props) => (
  <svg
    {...props}
    xmlns="http://www.w3.org/2000/svg"
    width="24"
    height="24"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    strokeWidth="2"
    strokeLinecap="round"
    strokeLinejoin="round"
  >
    <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
    <polyline points="17 8 12 3 7 8" />
    <line x1="12" x2="12" y1="3" y2="15" />
  </svg>
);

export default UploadIcon;
```

### MediaUpload/NoFileSelected.tsx

```tsx
import UploadIcon from "./UploadIcon";

const NoFileSelected = ({ onClick }) => (
  <div
    className="absolute inset-0 flex flex-col items-center justify-center space-y-2 text-muted-foreground cursor-pointer"
    onClick={onClick}
  >
    <UploadIcon className="w-8 h-8" />
    <p className="text-sm font-medium">click to upload</p>
  </div>
);

export default NoFileSelected;
```

### MediaUpload/AudioPlayer

```tsx
import { getIpfsLink } from "@/lib/utils";
import { useZoraCreateProvider } from "@/providers/ZoraCreateProvider";
import { useRef, useState } from "react";
import { Button } from "../ui/button";
import { Pause, Play } from "lucide-react";
import { Slider } from "../ui/slider";

const AudioPlayer = ({ onClick }) => {
  const { imageUri, animationUri } = useZoraCreateProvider();
  const [isPlaying, setIsPlaying] = useState(false);
  const [progress, setProgress] = useState(0);
  const audioRef = useRef<HTMLAudioElement>(null);

  const togglePlayPause = () => {
    if (audioRef.current) {
      if (isPlaying) {
        audioRef.current.pause();
      } else {
        audioRef.current.play();
      }
      setIsPlaying(!isPlaying);
    }
  };

  const handleTimeUpdate = () => {
    if (audioRef.current) {
      const progress =
        (audioRef.current.currentTime / audioRef.current.duration) * 100;
      setProgress(progress);
    }
  };

  const handleSliderChange = (value: number[]) => {
    if (audioRef.current) {
      const time = (value[0] / 100) * audioRef.current.duration;
      audioRef.current.currentTime = time;
      setProgress(value[0]);
    }
  };

  return (
    <div className="w-full bg-white rounded-lg shadow-lg overflow-hidden">
      <div className="relative" onClick={onClick}>
        {/* eslint-disable-next-line @next/next/no-img-element */}
        <img
          src={imageUri ? getIpfsLink(imageUri) : ""}
          alt="Audio cover"
          className="w-full h-auto cursor-pointer"
        />
      </div>
      <div className="p-4">
        <audio
          ref={audioRef}
          src={getIpfsLink(animationUri)}
          onTimeUpdate={handleTimeUpdate}
        />
        <div className="flex justify-center mb-4">
          <Button
            variant="ghost"
            size="icon"
            onClick={togglePlayPause}
            className="text-primary hover:text-primary-dark"
          >
            {isPlaying ? (
              <Pause className="h-8 w-8" />
            ) : (
              <Play className="h-8 w-8" />
            )}
          </Button>
        </div>
        <Slider
          value={[progress]}
          onValueChange={handleSliderChange}
          max={100}
          step={1}
          className="w-full bg-black"
        />
      </div>
    </div>
  );
};

export default AudioPlayer;
```

### MediaUpload/MediaUpload

```tsx
import { useZoraCreateProvider } from "@/providers/ZoraCreateProvider";
import useFileUpload from "@/hooks/useFileUpload";
import { cn } from "@/lib/utils";
import Spinner from "@/components/ui/spinner";
import { getIpfsLink } from "@/lib/utils";
import { useRef } from "react";
import NoFileSelected from "./NoFileSelected";
import AudioPlayer from "./AudioPlayer";
import Image from "next/image";

const MediaUpload = () => {
  const { imageUri, animationUri, mimeType } = useZoraCreateProvider();
  const { fileUpload, loading, error, blurImageUrl } = useFileUpload();
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

    if (mimeType.includes("audio")) {
      return <AudioPlayer onClick={handleImageClick} />;
    }

    if (mimeType.includes("video")) {
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
            layout="fill"
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
        />
        {renderMedia()}
      </div>
      {error && <p className="text-red-500 text-sm mt-2">{error}</p>}
    </div>
  );
};

export default MediaUpload;
```

### CreatePage/LoginButton

```tsx
"use client";

import { useAccount, useDisconnect } from "wagmi";
import useConnectWallet from "@/hooks/useConnectWallet";
import { Button } from "@/components/ui/button";

export default function LoginButton() {
  const { status } = useAccount();
  const { connectWallet } = useConnectWallet();
  const { disconnect } = useDisconnect();

  if (["connecting", "reconnecting"].includes(status))
    return <Button disabled>Loading...</Button>;

  if (status === "connected")
    return <Button onClick={() => disconnect()}>Disconnect</Button>;

  return <Button onClick={connectWallet}>Connect</Button>;
}
```

### CreatePage/Title

```tsx
import { useZoraCreateProvider } from "@/providers/ZoraCreateProvider";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

const Title = () => {
  const { name, setName } = useZoraCreateProvider();

  return (
    <div className="flex flex-col items-start w-full">
      <Label htmlFor="title">Title</Label>
      <Input
        id="title"
        value={name}
        onChange={(e) => setName(e.target.value)}
        className="border border-black py-7 text-xl"
      />
    </div>
  );
};

export default Title;
```

### CreatePage/CreateButton

```tsx
"use client";

import { Button } from "@/components/ui/button";
import { useZoraCreateProvider } from "@/providers/ZoraCreateProvider";

const CreateButton = ({ children }: any) => {
  const { create } = useZoraCreateProvider();

  return (
    <Button
      onClick={() => create()}
      className="bg-white text-black p-3 transform hover:scale-105 transition-transform duration-150 hover:shadow-lg"
    >
      {children}
    </Button>
  );
};

export default CreateButton;
```

### CreatePage/CreatePage

```tsx
"use client";

import { useAccount } from "wagmi";
import { useZoraCreateProvider } from "@/providers/ZoraCreateProvider";
import Spinner from "@/components/ui/spinner";
import MediaUpload from "@/components/MediaUpload/MediaUpload";
import LoginButton from "./LoginButton";
import Title from "./Title";
import CreateButton from "./CreateButton";

export default function CreatePage() {
  const { creating, name } = useZoraCreateProvider();
  const { address } = useAccount();

  if (creating) {
    return (
      <>
        <Spinner />
        <span>Creating Post!</span>
      </>
    );
  }

  return (
    <div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
      <div className="mt-8 md:flex md:space-x-8">
        <div className="md:w-1/2 flex flex-col items-center gap-5">
          <MediaUpload />
        </div>
        {name && (
          <div className="mt-4 md:mt-0 md:w-1/2 flex flex-col items-center gap-3">
            <div className="w-full flex flex-col items-start gap-4">
              <Title />
            </div>
            {address ? <CreateButton /> : <LoginButton />}
          </div>
        )}
      </div>
    </div>
  );
}
```

Finally, wrap your application with the providers:

### layout

```tsx
import { WagmiProvider } from "@/providers/WagmiProvider";
import { ZoraCreateProvider } from "@/providers/ZoraCreateProvider";

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

## Libs

### utils

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

### consts

```tsx
import { base } from "wagmi/chains";
// Wagmi
export const CHAIN = base;
export const CHAIN_ID = CHAIN.id;
// Zora
export const REFERRAL_RECIPIENT = "0x749B7b7A6944d72266Be9500FC8C221B6A7554Ce";
// IPFS
export const ONE_MB = 1024 * 1024;
export const MAX_FILE_SIZE = 5 * ONE_MB;
```

### zora/getSalesConfig

```tsx
import {
  FixedPriceParamsType,
  TimedSaleParamsType,
} from "@zoralabs/protocol-sdk";

const getSalesConfig = (saleStrategy: string) => {
  const timedSaleConfig = {
    type: "timed",
    erc20Name: "CC0 Music",
    erc20Symbol: "CC0",
  } as TimedSaleParamsType;
  const fixedPriceSaleConfig = {
    type: "fixedPrice",
    pricePerToken: BigInt(1),
  } as FixedPriceParamsType;
  return saleStrategy === "ZoraTimedSaleStrategy"
    ? timedSaleConfig
    : fixedPriceSaleConfig;
};

export default getSalesConfig;
```

### ipfs/uploadJson

```tsx
import { IPFSUploadResponse, uploadFile } from "./uploadFile";

export async function uploadJson(json: object): Promise<IPFSUploadResponse> {
  const jsonString = JSON.stringify(json);
  const file = new File([jsonString], "upload.json", {
    type: "application/json",
  });
  return await uploadFile(file);
}
```

### ipfs/saveFile

```tsx
const saveFile = async (data: any, jwt?: string) => {
  const response = await fetch(
    "https://api.pinata.cloud/pinning/pinFileToIPFS",
    {
      method: "POST",
      headers: {
        Authorization: `Bearer ${jwt ?? process.env.PINATA_JWT}`,
      },
      body: data,
    }
  );
  const { IpfsHash } = await response.json();
  return IpfsHash;
};

export default saveFile;
```

### ipfs/uploadFile

```tsx
import saveFile from "./saveFile";

export type IPFSUploadResponse = {
  cid: string;
  uri: string;
};

export const uploadFile = async (
  file: File,
  jwt?: string
): Promise<IPFSUploadResponse> => {
  try {
    console.log("uploadingFile...");
    const data = new FormData();
    console.log("data", data);

    data.set("file", file);
    console.log("file", file);

    let cid: any;
    console.log("jwt", jwt);
    console.log("data", data);
    if (jwt) {
      cid = await saveFile(data, jwt);
    } else {
      const res = await fetch("/api/ipfs", { method: "POST", body: data });
      const json = await res.json();
      cid = json.cid;
    }

    return { cid, uri: `ipfs://${cid}` };
  } catch (error) {
    console.error(error);
    return { cid: "", uri: "" };
  }
};
```

### ipfs/generatePinataJWT

```tsx
const KEY_RESTRICTIONS = {
  maxUses: 1,
  keyName: "Signed Upload JWT",
  permissions: {
    endpoints: {
      data: {
        pinList: false,
        userPinnedDataTotal: false,
      },
      pinning: {
        pinFileToIPFS: true,
        pinJSONToIPFS: true,
        pinJobs: false,
        unpin: false,
        userPinPolicy: false,
      },
    },
  },
};

async function generatePinataJWT() {
  const res = await fetch("https://api.pinata.cloud/users/generateApiKey", {
    method: "POST",
    headers: {
      accept: "application/json",
      "content-type": "application/json",
      authorization: `Bearer ${process.env.PINATA_JWT}`,
    },
    body: JSON.stringify(KEY_RESTRICTIONS),
  });
  const data = await res.json();

  if (!res.ok) throw data ?? { message: res.statusText };

  return { JWT: data?.JWT };
}

export default generatePinataJWT;
```

## Basic Usage

```tsx
import { CreatePage } from "@/components/create-page";

export default function Create() {
  return <CreatePage />;
}
```

## Component API

The `<CreatePage />` component does not accept any props.

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
