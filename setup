#!/bin/bash

# Create directories
mkdir -p app/api/onfido/applicant
mkdir -p app/api/onfido/token
mkdir -p app/onboarding
mkdir -p components/onboarding
mkdir -p components/ui
mkdir -p context
mkdir -p lib

# Create app files
cat > app/layout.tsx << 'EOL'
import type React from "react"
import "./globals.css"
import type { Metadata } from "next"
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ["latin"] })

export const metadata: Metadata = {
  title: "NEOMI - Blockchain Onboarding",
  description: "Simplified blockchain onboarding experience for Web3",
  metadataBase: new URL("https://neomi.wtf"),
  openGraph: {
    title: "NEOMI - Blockchain Onboarding",
    description: "Simplified blockchain onboarding experience for Web3",
    url: "https://neomi.wtf/onboarding",
    siteName: "NEOMI",
    images: [
      {
        url: "https://neomi.wtf/og-image.png",
        width: 1200,
        height: 630,
        alt: "NEOMI Blockchain Onboarding",
      },
    ],
    locale: "en_US",
    type: "website",
  },
  twitter: {
    card: "summary_large_image",
    title: "NEOMI - Blockchain Onboarding",
    description: "Simplified blockchain onboarding experience for Web3",
    images: ["https://neomi.wtf/og-image.png"],
  },
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  )
}
EOL

cat > app/page.tsx << 'EOL'
import { redirect } from "next/navigation"

export default function Home() {
  redirect("/onboarding")
  return null
}
EOL

cat > app/globals.css << 'EOL'
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --foreground-rgb: 0, 0, 0;
  --background-start-rgb: 240, 240, 245;
  --background-end-rgb: 255, 255, 255;
}

@media (prefers-color-scheme: dark) {
  :root {
    --foreground-rgb: 255, 255, 255;
    --background-start-rgb: 10, 10, 20;
    --background-end-rgb: 0, 0, 0;
  }
}

body {
  color: rgb(var(--foreground-rgb));
  background: linear-gradient(to bottom, rgb(var(--background-start-rgb)), rgb(var(--background-end-rgb)));
  min-height: 100vh;
}
EOL

cat > app/providers.tsx << 'EOL'
"use client"

import type React from "react"

import { WalletProvider } from "@/context/wallet-context"
import { OnboardingProvider } from "@/context/onboarding-context"

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <WalletProvider>
      <OnboardingProvider>{children}</OnboardingProvider>
    </WalletProvider>
  )
}
EOL

cat > app/onboarding/page.tsx << 'EOL'
import { OnboardingWizard } from "@/components/onboarding-wizard"
import { Providers } from "@/app/providers"

export default function OnboardingPage() {
  return (
    <main className="min-h-screen bg-gradient-to-b from-gray-50 to-gray-100 py-8 px-4">
      <Providers>
        <OnboardingWizard />
      </Providers>
    </main>
  )
}
EOL

# Create API routes
cat > app/api/onfido/applicant/route.ts << 'EOL'
import { NextResponse } from "next/server"

export async function POST(request: Request) {
  try {
    const { firstName, lastName, email } = await request.json()

    // Get Onfido API key from environment variables
    const apiKey = process.env.ONFIDO_API_KEY

    if (!apiKey) {
      console.error("Onfido API key not configured")
      return NextResponse.json({ error: "Onfido API key not configured" }, { status: 500 })
    }

    console.log("Creating Onfido applicant:", { firstName, lastName, email })

    // Create an Onfido applicant
    const response = await fetch("https://api.onfido.com/v3/applicants", {
      method: "POST",
      headers: {
        Authorization: `Token token=${apiKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        first_name: firstName,
        last_name: lastName,
        email: email,
      }),
    })

    if (!response.ok) {
      const error = await response.json()
      console.error("Onfido API error:", error)
      return NextResponse.json({ error: "Failed to create Onfido applicant" }, { status: response.status })
    }

    const data = await response.json()
    return NextResponse.json(data)
  } catch (error) {
    console.error("Error creating Onfido applicant:", error)
    return NextResponse.json({ error: "Internal server error" }, { status: 500 })
  }
}
EOL

cat > app/api/onfido/token/route.ts << 'EOL'
import { NextResponse } from "next/server"

export async function POST(request: Request) {
  try {
    const { applicantId } = await request.json()

    // Get Onfido API key from environment variables
    const apiKey = process.env.ONFIDO_API_KEY

    if (!apiKey) {
      console.error("Onfido API key not configured")
      return NextResponse.json({ error: "Onfido API key not configured" }, { status: 500 })
    }

    console.log("Creating Onfido SDK token for applicant:", applicantId)

    // Create an Onfido SDK token
    const response = await fetch("https://api.onfido.com/v3/sdk_token", {
      method: "POST",
      headers: {
        Authorization: `Token token=${apiKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        applicant_id: applicantId,
        referrer: "https://neomi.wtf/*",
      }),
    })

    if (!response.ok) {
      const error = await response.json()
      console.error("Onfido API error:", error)
      return NextResponse.json({ error: "Failed to create Onfido SDK token" }, { status: response.status })
    }

    const data = await response.json()
    return NextResponse.json(data)
  } catch (error) {
    console.error("Error generating Onfido token:", error)
    return NextResponse.json({ error: "Internal server error" }, { status: 500 })
  }
}
EOL

# Create context files
cat > context/wallet-context.tsx << 'EOL'
"use client"

import type React from "react"

import { createContext, useContext, useState, useEffect } from "react"

type WalletContextType = {
  address: string | null
  ensName: string | null
  isConnected: boolean
  balance: string
  connect: () => void
  disconnect: () => void
  isLoading: boolean
}

const WalletContext = createContext<WalletContextType>({
  address: null,
  ensName: null,
  isConnected: false,
  balance: "0.0000",
  connect: () => {},
  disconnect: () => {},
  isLoading: false,
})

export function WalletProvider({ children }: { children: React.ReactNode }) {
  const [address, setAddress] = useState<string | null>(null)
  const [ensName, setEnsName] = useState<string | null>(null)
  const [isConnected, setIsConnected] = useState(false)
  const [balance, setBalance] = useState("0.0000")
  const [isLoading, setIsLoading] = useState(false)
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)

    // Check if we have stored wallet info in localStorage
    try {
      const storedAddress = localStorage.getItem("walletAddress")
      const storedEnsName = localStorage.getItem("walletEnsName")
      const storedBalance = localStorage.getItem("walletBalance")

      if (storedAddress) {
        setAddress(storedAddress)
        setEnsName(storedEnsName)
        setBalance(storedBalance || "1.2345")
        setIsConnected(true)
      }
    } catch (error) {
      console.error("Error accessing localStorage:", error)
    }
  }, [])

  const connect = () => {
    setIsLoading(true)

    // Simulate connection delay
    setTimeout(() => {
      const mockAddress = "0x71C7656EC7ab88b098defB751B7401B5f6d8976F"
      const mockEnsName = "neomi.eth"
      const mockBalance = "1.2345"

      setAddress(mockAddress)
      setEnsName(mockEnsName)
      setBalance(mockBalance)
      setIsConnected(true)

      try {
        localStorage.setItem("walletAddress", mockAddress)
        localStorage.setItem("walletEnsName", mockEnsName)
        localStorage.setItem("walletBalance", mockBalance)
      } catch (error) {
        console.error("Error setting localStorage:", error)
      }

      setIsLoading(false)
    }, 1000)
  }

  const disconnect = () => {
    setAddress(null)
    setEnsName(null)
    setBalance("0.0000")
    setIsConnected(false)

    try {
      localStorage.removeItem("walletAddress")
      localStorage.removeItem("walletEnsName")
      localStorage.removeItem("walletBalance")
    } catch (error) {
      console.error("Error removing from localStorage:", error)
    }
  }

  return (
    <WalletContext.Provider
      value={{
        address,
        ensName,
        isConnected,
        balance,
        connect,
        disconnect,
        isLoading,
      }}
    >
      {mounted ? children : null}
    </WalletContext.Provider>
  )
}

export function useWallet() {
  return useContext(WalletContext)
}
EOL

cat > context/onboarding-context.tsx << 'EOL'
"use client"

import type React from "react"

import { createContext, useContext, useState, useEffect } from "react"
import { useWallet } from "./wallet-context"

type OnboardingStep = "welcome" | "profile" | "kyc" | "education" | "transaction" | "complete"

type UserData = {
  username: string
  email: string
  walletAddress: string | null
  kycVerified: boolean
  onfidoApplicantId?: string
  ensName?: string | null
}

type OnboardingContextType = {
  currentStep: OnboardingStep
  setCurrentStep: (step: OnboardingStep) => void
  userData: UserData
  updateUserData: (data: Partial<UserData>) => void
  isComplete: boolean
  markComplete: () => void
}

const OnboardingContext = createContext<OnboardingContextType>({
  currentStep: "welcome",
  setCurrentStep: () => {},
  userData: {
    username: "",
    email: "",
    walletAddress: null,
    kycVerified: false,
  },
  updateUserData: () => {},
  isComplete: false,
  markComplete: () => {},
})

export function OnboardingProvider({ children }: { children: React.ReactNode }) {
  const { address, ensName } = useWallet()
  const [currentStep, setCurrentStep] = useState<OnboardingStep>("welcome")
  const [userData, setUserData] = useState<UserData>({
    username: "",
    email: "",
    walletAddress: null,
    kycVerified: false,
    ensName: null,
  })
  const [isComplete, setIsComplete] = useState(false)
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)

    // Try to load saved progress from localStorage
    try {
      const savedProgress = localStorage.getItem("onboardingProgress")
      const savedUserData = localStorage.getItem("onboardingUserData")

      if (savedProgress) {
        setCurrentStep(savedProgress as OnboardingStep)
      }

      if (savedUserData) {
        setUserData(JSON.parse(savedUserData))
      }

      if (savedProgress === "complete") {
        setIsComplete(true)
      }
    } catch (error) {
      console.error("Error loading progress:", error)
    }
  }, [])

  // Update wallet address and ENS name when they change
  useEffect(() => {
    if (mounted && address) {
      setUserData((prev) => ({
        ...prev,
        walletAddress: address,
        ensName: ensName,
        username: prev.username || (ensName ? ensName.split(".")[0] : ""),
      }))
    }
  }, [address, ensName, mounted])

  const updateUserData = (data: Partial<UserData>) => {
    setUserData((prev) => {
      const updated = { ...prev, ...data }

      // Save to localStorage
      try {
        localStorage.setItem("onboardingUserData", JSON.stringify(updated))
      } catch (error) {
        console.error("Error saving user data:", error)
      }

      return updated
    })
  }

  const markComplete = () => {
    setIsComplete(true)
    setCurrentStep("complete")

    // Save to localStorage
    try {
      localStorage.setItem("onboardingProgress", "complete")
    } catch (error) {
      console.error("Error saving progress:", error)
    }
  }

  // Save current step to localStorage when it changes
  useEffect(() => {
    if (mounted) {
      try {
        localStorage.setItem("onboardingProgress", currentStep)
      } catch (error) {
        console.error("Error saving progress:", error)
      }
    }
  }, [currentStep, mounted])

  return (
    <OnboardingContext.Provider
      value={{
        currentStep,
        setCurrentStep,
        userData,
        updateUserData,
        isComplete,
        markComplete,
      }}
    >
      {mounted ? children : null}
    </OnboardingContext.Provider>
  )
}

export function useOnboarding() {
  return useContext(OnboardingContext)
}
EOL

# Create lib files
cat > lib/utils.ts << 'EOL'
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

export function formatAddress(address: string | null): string {
  if (!address) return ""
  return `${address.slice(0, 6)}...${address.slice(-4)}`
}

export function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}

export function isValidEthAddress(address: string): boolean {
  return /^0x[a-fA-F0-9]{40}$/.test(address)
}
EOL

# Create UI components
cat > components/ui/button.tsx << 'EOL'
import { cn } from "@/lib/utils"
import type React from "react"

export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "default" | "destructive" | "outline" | "secondary" | "ghost" | "link"
  size?: "default" | "sm" | "lg" | "icon"
}

export function Button({ className, variant = "default", size = "default", ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        "inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none",
        {
          "bg-blue-600 text-white hover:bg-blue-700 dark:bg-blue-600 dark:hover:bg-blue-700": variant === "default",
          "bg-red-600 text-white hover:bg-red-700 dark:bg-red-600 dark:hover:bg-red-700": variant === "destructive",
          "border border-gray-300 bg-white hover:bg-gray-100 dark:border-gray-700 dark:bg-gray-950 dark:hover:bg-gray-900":
            variant === "outline",
          "bg-gray-100 text-gray-900 hover:bg-gray-200 dark:bg-gray-800 dark:text-gray-50 dark:hover:bg-gray-700":
            variant === "secondary",
          "hover:bg-gray-100 hover:text-gray-900 dark:hover:bg-gray-800 dark:hover:text-gray-50": variant === "ghost",
          "text-blue-600 underline-offset-4 hover:underline dark:text-blue-500": variant === "link",
          "h-10 px-4 py-2": size === "default",
          "h-9 rounded-md px-3": size === "sm",
          "h-11 rounded-md px-8": size === "lg",
          "h-10 w-10": size === "icon",
        },
        className,
      )}
      {...props}
    />
  )
}
EOL

cat > components/ui/card.tsx << 'EOL'
import { cn } from "@/lib/utils"
import type React from "react"

export interface CardProps extends React.HTMLAttributes<HTMLDivElement> {}

export function Card({ className, ...props }: CardProps) {
  return (
    <div
      className={cn(
        "rounded-lg border border-gray-200 bg-white shadow-sm dark:border-gray-800 dark:bg-gray-950",
        className,
      )}
      {...props}
    />
  )
}

export interface CardHeaderProps extends React.HTMLAttributes<HTMLDivElement> {}

export function CardHeader({ className, ...props }: CardHeaderProps) {
  return <div className={cn("px-6 pt-6", className)} {...props} />
}

export interface CardTitleProps extends React.HTMLAttributes<HTMLHeadingElement> {}

export function CardTitle({ className, ...props }: CardTitleProps) {
  return <h3 className={cn("text-xl font-semibold", className)} {...props} />
}

export interface CardDescriptionProps extends React.HTMLAttributes<HTMLParagraphElement> {}

export function CardDescription({ className, ...props }: CardDescriptionProps) {
  return <p className={cn("text-sm text-gray-500 dark:text-gray-400", className)} {...props} />
}

export interface CardContentProps extends React.HTMLAttributes<HTMLDivElement> {}

export function CardContent({ className, ...props }: CardContentProps) {
  return <div className={cn("px-6 py-4", className)} {...props} />
}

export interface CardFooterProps extends React.HTMLAttributes<HTMLDivElement> {}

export function CardFooter({ className, ...props }: CardFooterProps) {
  return <div className={cn("flex items-center px-6 pb-6", className)} {...props} />
}
EOL

# Create onboarding components
cat > components/onboarding-wizard.tsx << 'EOL'
"use client"

import { useEffect, useState } from "react"
import { useWallet } from "@/context/wallet-context"
import { useOnboarding } from "@/context/onboarding-context"
import { WalletConnect } from "./wallet-connect"
import { WelcomeStep } from "./onboarding/welcome-step"
import { ProfileStep } from "./onboarding/profile-step"
import { KycStep } from "./onboarding/kyc-step"
import { EducationStep } from "./onboarding/education-step"
import { TransactionStep } from "./onboarding/transaction-step"
import { CompleteStep } from "./onboarding/complete-step"

export function OnboardingWizard() {
  const { isConnected } = useWallet()
  const { currentStep } = useOnboarding()
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="animate-pulse text-center">
          <div className="w-16 h-16 mx-auto mb-4 rounded-full bg-blue-100"></div>
          <p className="text-gray-500">Loading...</p>
        </div>
      </div>
    )
  }

  return (
    <div className="max-w-4xl mx-auto">
      <div className="text-center mb-8">
        <div className="flex justify-center mb-6">
          <div className="relative w-40 h-12">
            <h1 className="text-3xl font-bold">NEOMI</h1>
          </div>
        </div>

        {isConnected && (
          <div className="mb-8">
            <div className="flex justify-between items-center mb-4">
              <h2 className="text-2xl font-bold">Blockchain Onboarding</h2>
              <div className="text-sm text-gray-500">Step {getStepNumber(currentStep)} of 6</div>
            </div>
            <div className="w-full bg-gray-200 h-2 rounded-full overflow-hidden">
              <div
                className="bg-blue-600 h-full transition-all duration-300 ease-in-out"
                style={{
                  width: `${(getStepNumber(currentStep) / 6) * 100}%`,
                }}
              />
            </div>
          </div>
        )}
      </div>

      <div className="max-w-md mx-auto">
        {!isConnected ? (
          <WalletConnect />
        ) : (
          <>
            {currentStep === "welcome" && <WelcomeStep />}
            {currentStep === "profile" && <ProfileStep />}
            {currentStep === "kyc" && <KycStep />}
            {currentStep === "education" && <EducationStep />}
            {currentStep === "transaction" && <TransactionStep />}
            {currentStep === "complete" && <CompleteStep />}
          </>
        )}
      </div>
    </div>
  )
}

function getStepNumber(step: string): number {
  const steps = ["welcome", "profile", "kyc", "education", "transaction", "complete"]
  return steps.indexOf(step) + 1
}
EOL

cat > components/wallet-connect.tsx << 'EOL'
"use client"

import { useEffect, useState } from "react"
import { useWallet } from "@/context/wallet-context"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"

export function WalletConnect() {
  const { isConnected, connect, disconnect, address, ensName, isLoading } = useWallet()
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) {
    return (
      <Card>
        <CardHeader>
          <CardTitle>Connect Your Wallet</CardTitle>
          <CardDescription>Loading wallet connection...</CardDescription>
        </CardHeader>
        <CardContent className="flex justify-center py-6">
          <div className="animate-pulse w-full max-w-xs">
            <div className="h-10 bg-gray-200 rounded w-full"></div>
          </div>
        </CardContent>
      </Card>
    )
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>Connect Your Wallet</CardTitle>
        <CardDescription>Connect your wallet to get started with NEOMI blockchain onboarding</CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        {isConnected ? (
          <div className="text-center">
            <p className="mb-2">Connected with</p>
            {ensName && <p className="text-lg font-medium mb-2">{ensName}</p>}
            <code className="bg-gray-100 p-2 rounded-md block mb-4 break-all text-sm">{address}</code>
            <Button variant="destructive" onClick={disconnect}>
              Disconnect
            </Button>
          </div>
        ) : (
          <div className="space-y-4">
            <p className="text-center text-sm text-gray-500">
              To continue with the onboarding process, please connect your Ethereum wallet.
            </p>
            <Button className="w-full" onClick={connect} disabled={isLoading}>
              {isLoading ? "Connecting..." : "Connect Wallet"}
            </Button>
          </div>
        )}
      </CardContent>
    </Card>
  )
}
EOL

cat > components/onboarding/welcome-step.tsx << 'EOL'
"use client"

import { useOnboarding } from "@/context/onboarding-context"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"

export function WelcomeStep() {
  const { setCurrentStep } = useOnboarding()

  return (
    <Card>
      <CardHeader>
        <CardTitle>Welcome to NEOMI</CardTitle>
        <CardDescription>Your journey into blockchain starts here</CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <p className="text-sm">
          NEOMI is designed to make your blockchain experience simple and intuitive. We'll guide you through the basics
          and help you get started with confidence.
        </p>
        <p className="text-sm">In this onboarding process, you'll:</p>
        <ul className="list-disc pl-5 space-y-2 text-sm">
          <li>Set up your profile</li>
          <li>Complete identity verification (KYC)</li>
          <li>Learn blockchain basics</li>
          <li>Complete your first transaction</li>
        </ul>
      </CardContent>
      <CardFooter>
        <Button className="w-full" onClick={() => setCurrentStep("profile")}>
          Get Started
        </Button>
      </CardFooter>
    </Card>
  )
}
EOL

cat > components/onboarding/profile-step.tsx << 'EOL'
"use client"

import type React from "react"

import { useState, useEffect } from "react"
import { useWallet } from "@/context/wallet-context"
import { useOnboarding } from "@/context/onboarding-context"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { isValidEmail } from "@/lib/utils"

export function ProfileStep() {
  const { address, ensName } = useWallet()
  const { userData, updateUserData, setCurrentStep } = useOnboarding()
  const [username, setUsername] = useState(userData.username || "")
  const [email, setEmail] = useState(userData.email || "")
  const [errors, setErrors] = useState({ username: "", email: "" })
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)

    // If we have an ENS name and no username yet, use the ENS name
    if (ensName && !username) {
      setUsername(ensName.split(".")[0])
    }
  }, [ensName, username])

  const validate = () => {
    const newErrors = { username: "", email: "" }
    let isValid = true

    if (!username.trim()) {
      newErrors.username = "Username is required"
      isValid = false
    }

    if (!email.trim()) {
      newErrors.email = "Email is required"
      isValid = false
    } else if (!isValidEmail(email)) {
      newErrors.email = "Please enter a valid email address"
      isValid = false
    }

    setErrors(newErrors)
    return isValid
  }

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()

    if (!validate()) return

    updateUserData({ username, email })
    setCurrentStep("kyc")
  }

  if (!mounted) {
    return (
      <Card>
        <CardHeader>
          <CardTitle>Create Your Profile</CardTitle>
          <CardDescription>Loading...</CardDescription>
        </CardHeader>
      </Card>
    )
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>Create Your Profile</CardTitle>
        <CardDescription>Tell us a little about yourself</CardDescription>
      </CardHeader>
      <CardContent>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label htmlFor="username" className="block text-sm font-medium text-gray-700 mb-1">
              Username
            </label>
            <input
              id="username"
              type="text"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              className="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            {errors.username && <p className="mt-1 text-sm text-red-600">{errors.username}</p>}
            {ensName && <p className="text-xs text-gray-500 mt-1">Pre-filled from your ENS name: {ensName}</p>}
          </div>

          <div>
            <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
              Email
            </label>
            <input
              id="email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              className="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            {errors.email && <p className="mt-1 text-sm text-red-600">{errors.email}</p>}
          </div>

          

          <div>
            <label htmlFor="wallet" className="block text-sm font-medium text-gray-700 mb-1">
              Wallet Address
            </label>
            <input
              id="wallet"
              type="text"
              value={address || ""}
              className="w-full p-2 border border-gray-300 rounded-md bg-gray-50"
              disabled
            />
            {ensName && <p className="text-xs text-gray-500 mt-1">ENS Name: {ensName}</p>}
          </div>
        </form>
      </CardContent>
      <CardFooter className="flex justify-between">
        <Button variant="outline" onClick={() => setCurrentStep("welcome")}>
          Back
        </Button>
        <Button onClick={handleSubmit}>Continue</Button>
      </CardFooter>
    </Card>
  )
}
EOL

cat > components/onboarding/kyc-step.tsx << 'EOL'
"use client"

import { useEffect, useState } from "react"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { useOnboarding } from "@/context/onboarding-context"
import Script from "next/script"

// Define the Onfido global object type
declare global {
  interface Window {
    Onfido: any
  }
}

export function KycStep() {
  const { userData, updateUserData, setCurrentStep } = useOnboarding()
  const [isLoading, setIsLoading] = useState(true)
  const [isComplete, setIsComplete] = useState(false)
  const [applicantId, setApplicantId] = useState<string | null>(null)
  const [sdkToken, setSdkToken] = useState<string | null>(null)
  const [onfidoInstance, setOnfidoInstance] = useState<any>(null)
  const [scriptLoaded, setScriptLoaded] = useState(false)
  const [mounted, setMounted] = useState(false)
  const [error, setError] = useState<string | null>(null)

  // Prevent hydration errors
  useEffect(() => {
    setMounted(true)
  }, [])

  // Create an Onfido applicant when the component mounts
  useEffect(() => {
    if (!mounted) return

    async function createApplicant() {
      try {
        setIsLoading(true)
        // Extract first and last name from username (in a real app, collect these separately)
        const nameParts = userData.username.split(" ")
        const firstName = nameParts[0] || userData.username
        const lastName = nameParts.length > 1 ? nameParts.slice(1).join(" ") : ""

        console.log("Creating applicant with:", { firstName, lastName, email: userData.email })

        const response = await fetch("/api/onfido/applicant", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            firstName,
            lastName,
            email: userData.email,
          }),
        })

        if (!response.ok) {
          throw new Error(`Failed to create applicant: ${response.status}`)
        }

        const data = await response.json()
        console.log("Applicant created:", data)

        setApplicantId(data.id)

        // Store the applicant ID in the user data
        updateUserData({
          onfidoApplicantId: data.id,
        })
      } catch (error) {
        console.error("Error creating applicant:", error)
        setError("Failed to initialize identity verification. Please try again.")
      } finally {
        setIsLoading(false)
      }
    }

    if (userData.email && userData.username) {
      createApplicant()
    }
  }, [userData.email, userData.username, updateUserData, userData, mounted])

  // Get an SDK token once we have an applicant ID
  useEffect(() => {
    if (!mounted || !applicantId || !scriptLoaded) return

    async function getSDKToken() {
      try {
        console.log("Getting SDK token for applicant:", applicantId)

        const response = await fetch("/api/onfido/token", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            applicantId,
          }),
        })

        if (!response.ok) {
          throw new Error(`Failed to get SDK token: ${response.status}`)
        }

        const data = await response.json()
        console.log("SDK token received")

        setSdkToken(data.token)
      } catch (error) {
        console.error("Error getting SDK token:", error)
        setError("Failed to initialize verification flow. Please try again.")
      }
    }

    if (applicantId) {
      getSDKToken()
    }
  }, [applicantId, scriptLoaded, mounted])

  // Initialize the Onfido SDK once we have a token
  useEffect(() => {
    if (!mounted || !sdkToken || !window.Onfido || !scriptLoaded) return

    try {
      console.log("Initializing Onfido SDK with token")

      const onfido = window.Onfido.init({
        token: sdkToken,
        containerId: "onfido-mount",
        onComplete: (data: any) => {
          console.log("Onfido verification complete")
          setIsComplete(true)

          // In a production app, you would check the verification status via your backend
          updateUserData({
            kycVerified: true,
          })
        },
        onError: (error: any) => {
          console.error("Onfido error:", error)
          setError(error.message || "An error occurred during verification. Please try again.")
        },
        steps: [
          {
            type: "welcome",
            options: {
              title: "Identity Verification",
              subtitle: "Verify your identity to continue with NEOMI",
            },
          },
          {
            type: "document",
            options: {
              documentTypes: {
                passport: true,
                driving_licence: true,
                national_identity_card: true,
              },
            },
          },
          { type: "face", options: { type: "video" } },
          { type: "complete" },
        ],
        isModalOpen: true,
        singleDocumentMode: true,
      })

      setOnfidoInstance(onfido)
      setIsLoading(false)

      return () => {
        if (onfido) {
          onfido.tearDown()
        }
      }
    } catch (error) {
      console.error("Error initializing Onfido:", error)
      setError("Failed to initialize verification. Please try again.")
    }
  }, [sdkToken, updateUserData, scriptLoaded, mounted])

  const handleContinue = () => {
    setCurrentStep("education")
  }

  if (!mounted) {
    return (
      <Card>
        <CardHeader>
          <CardTitle>Identity Verification</CardTitle>
          <CardDescription>Loading verification system...</CardDescription>
        </CardHeader>
        <CardContent className="flex justify-center py-12">
          <div className="animate-spin h-8 w-8 border-4 border-blue-500 rounded-full border-t-transparent"></div>
        </CardContent>
      </Card>
    )
  }

  return (
    <>
      <Script
        src="https://assets.onfido.com/web-sdk-releases/latest/onfido.min.js"
        onLoad={() => {
          console.log("Onfido SDK script loaded")
          setScriptLoaded(true)
        }}
        onError={() => {
          console.error("Failed to load Onfido SDK")
          setError("Failed to load verification system. Please try again.")
        }}
        strategy="afterInteractive"
      />

      <Card>
        <CardHeader>
          <CardTitle>Identity Verification</CardTitle>
          <CardDescription>We need to verify your identity before you can proceed with transactions</CardDescription>
        </CardHeader>
        <CardContent>
          {error && (
            <div className="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded mb-4">
              <p className="text-sm">{error}</p>
              <Button variant="outline" size="sm" className="mt-2" onClick={() => window.location.reload()}>
                Try Again
              </Button>
            </div>
          )}

          {isLoading ? (
            <div className="flex flex-col items-center justify-center py-12">
              <div className="animate-spin h-8 w-8 border-4 border-blue-500 rounded-full border-t-transparent mb-4"></div>
              <p>Initializing verification...</p>
            </div>
          ) : isComplete ? (
            <div className="flex flex-col items-center justify-center py-8 space-y-4">
              <div className="rounded-full bg-green-100 p-3">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  className="h-8 w-8 text-green-600"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke="currentColor"
                >
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
                </svg>
              </div>
              <h3 className="text-xl font-medium text-center">Verification Submitted</h3>
              <p className="text-center text-gray-500">
                Your identity verification has been submitted successfully. You can now proceed to the next step.
              </p>
            </div>
          ) : (
            <div id="onfido-mount" className="min-h-[400px]"></div>
          )}
        </CardContent>
        <CardFooter className="flex justify-between">
          <Button variant="outline" onClick={() => setCurrentStep("profile")}>
            Back
          </Button>
          <Button onClick={handleContinue} disabled={!isComplete}>
            Continue
          </Button>
        </CardFooter>
      </Card>
    </>
  )
}
EOL

cat > components/onboarding/education-step.tsx << 'EOL'
"use client"

import { useState } from "react"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { useOnboarding } from "@/context/onboarding-context"

export function EducationStep() {
  const { setCurrentStep } = useOnboarding()
  const [activeTab, setActiveTab] = useState("blockchain")
  const [completedTabs, setCompletedTabs] = useState<string[]>([])

  const markTabComplete = (tab: string) => {
    if (!completedTabs.includes(tab)) {
      setCompletedTabs([...completedTabs, tab])
    }
  }

  const allTabsCompleted = ["blockchain", "wallet", "transactions"].every((tab) => completedTabs.includes(tab))

  return (
    <Card>
      <CardHeader>
        <CardTitle>Blockchain Basics</CardTitle>
        <CardDescription>Learn the essentials of blockchain technology</CardDescription>
      </CardHeader>
      <CardContent>
        <div className="mb-4">
          <div className="flex border-b">
            <button
              className={`px-4 py-2 font-medium text-sm ${
                activeTab === "blockchain"
                  ? "border-b-2 border-blue-500 text-blue-600"
                  : "text-gray-500 hover:text-gray-700"
              }`}
              onClick={() => setActiveTab("blockchain")}
            >
              Blockchain
              {completedTabs.includes("blockchain") && <span className="ml-2 text-green-500">✓</span>}
            </button>
            <button
              className={`px-4 py-2 font-medium text-sm ${
                activeTab === "wallet"
                  ? "border-b-2 border-blue-500 text-blue-600"
                  : "text-gray-500 hover:text-gray-700"
              }`}
              onClick={() => setActiveTab("wallet")}
            >
              Wallets
              {completedTabs.includes("wallet") && <span className="ml-2 text-green-500">✓</span>}
            </button>
            <button
              className={`px-4 py-2 font-medium text-sm ${
                activeTab === "transactions"
                  ? "border-b-2 border-blue-500 text-blue-600"
                  : "text-gray-500 hover:text-gray-700"
              }`}
              onClick={() => setActiveTab("transactions")}
            >
              Transactions
              {completedTabs.includes("transactions") && <span className="ml-2 text-green-500">✓</span>}
            </button>
          </div>
        </div>

        <div className="py-2">
          {activeTab === "blockchain" && (
            <div className="space-y-4">
              <h3 className="text-lg font-medium">What is Blockchain?</h3>
              <p className="text-sm">
                Blockchain is a distributed database that maintains a continuously growing list of records, called
                blocks, which are linked and secured using cryptography.
              </p>
              <p className="text-sm">Key features of blockchain include:</p>
              <ul className="list-disc pl-5 space-y-2 text-sm">
                <li>Decentralization - No single entity controls the network</li>
                <li>Transparency - All transactions are visible to participants</li>
                <li>Immutability - Once recorded, data cannot be altered</li>
              </ul>
              <Button onClick={() => markTabComplete("blockchain")} className="w-full mt-4">
                Mark as Complete
              </Button>
            </div>
          )}

          {activeTab === "wallet" && (
            <div className="space-y-4">
              <h3 className="text-lg font-medium">Crypto Wallets</h3>
              <p className="text-sm">
                A crypto wallet is a digital tool that allows you to store, send, and receive cryptocurrencies. It
                contains your private keys, which prove your ownership of digital assets.
              </p>
              <p className="text-sm">Types of wallets:</p>
              <ul className="list-disc pl-5 space-y-2 text-sm">
                <li>Hot wallets - Connected to the internet (like MetaMask)</li>
                <li>Cold wallets - Offline storage devices for enhanced security</li>
                <li>Custodial wallets - Managed by a third party</li>
              </ul>
              <Button onClick={() => markTabComplete("wallet")} className="w-full mt-4">
                Mark as Complete
              </Button>
            </div>
          )}

          {activeTab === "transactions" && (
            <div className="space-y-4">
              <h3 className="text-lg font-medium">Blockchain Transactions</h3>
              <p className="text-sm">
                Blockchain transactions are operations that change the state of the blockchain. They typically involve
                transferring assets between addresses.
              </p>
              <p className="text-sm">Transaction components:</p>
              <ul className="list-disc pl-5 space-y-2 text-sm">
                <li>Sender address - Where assets come from</li>
                <li>Recipient address - Where assets go to</li>
                <li>Value - Amount being transferred</li>
                <li>Gas fee - Cost to process the transaction</li>
              </ul>
              <Button onClick={() => markTabComplete("transactions")} className="w-full mt-4">
                Mark as Complete
              </Button>
            </div>
          )}
        </div>
      </CardContent>
      <CardFooter className="flex justify-between">
        <Button variant="outline" onClick={() => setCurrentStep("kyc")}>
          Back
        </Button>
        <Button onClick={() => setCurrentStep("transaction")} disabled={!allTabsCompleted}>
          Continue
        </Button>
      </CardFooter>
    </Card>
  )
}
EOL

cat > components/onboarding/transaction-step.tsx << 'EOL'
"use client"

import { useState } from "react"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { useOnboarding } from "@/context/onboarding-context"
import { useWallet } from "@/context/wallet-context"
import { isValidEthAddress } from "@/lib/utils"

export function TransactionStep() {
  const { setCurrentStep } = useOnboarding()
  const { balance } = useWallet()
  const [recipient, setRecipient] = useState("")
  const [amount, setAmount] = useState("")
  const [isSimulating, setIsSimulating] = useState(false)
  const [isPending, setIsPending] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [simulationResult, setSimulationResult] = useState<string | null>(null)

  const handleSimulate = () => {
    // Reset states
    setError(null)
    setSimulationResult(null)

    // Validate inputs
    if (!recipient) {
      setError("Please enter a recipient address")
      return
    }

    if (!isValidEthAddress(recipient) && !recipient.endsWith(".eth")) {
      setError("Please enter a valid Ethereum address or ENS name")
      return
    }

    if (!amount || Number.parseFloat(amount) <= 0) {
      setError("Please enter a valid amount")
      return
    }

    // Simulate transaction
    setIsSimulating(true)
    setTimeout(() => {
      setIsSimulating(false)
      setSimulationResult(
        `Transaction would cost approximately 0.0001 ETH in gas fees. Total cost: ${amount} ETH + 0.0001 ETH gas`,
      )
    }, 1500)
  }

  const handleSubmit = () => {
    // Reset error
    setError(null)

    // Validate inputs
    if (!recipient) {
      setError("Please enter a recipient address")
      return
    }

    if (!isValidEthAddress(recipient) && !recipient.endsWith(".eth")) {
      setError("Please enter a valid Ethereum address or ENS name")
      return
    }

    if (!amount || Number.parseFloat(amount) <= 0) {
      setError("Please enter a valid amount")
      return
    }

    // Simulate transaction submission
    setIsPending(true)
    setTimeout(() => {
      setIsPending(false)
      setCurrentStep("complete")
    }, 2000)
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>Your First Transaction</CardTitle>
        <CardDescription>Practice sending a transaction (simulation only)</CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        {error && (
          <div className="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded">
            <p className="text-sm">{error}</p>
          </div>
        )}

        {simulationResult && (
          <div className="bg-green-50 border border-green-200 text-green-700 px-4 py-3 rounded">
            <p className="text-sm">{simulationResult}</p>
          </div>
        )}

        <div className="p-4 bg-gray-100 rounded-md">
          <p className="text-sm font-medium">Your Balance</p>
          <p className="text-xl">{balance} ETH</p>
        </div>

        <div className="space-y-2">
          <label htmlFor="recipient" className="block text-sm font-medium text-gray-700">
            Recipient Address
          </label>
          <input
            id="recipient"
            placeholder="0x... or ENS name"
            value={recipient}
            onChange={(e) => setRecipient(e.target.value)}
            className="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div className="space-y-2">
          <label htmlFor="amount" className="block text-sm font-medium text-gray-700">
            Amount (ETH)
          </label>
          <input
            id="amount"
            type="number"
            step="0.0001"
            min="0"
            placeholder="0.01"
            value={amount}
            onChange={(e) => setAmount(e.target.value)}
            className="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div className="bg-yellow-50 p-4 rounded-md border border-yellow-200">
          <p className="text-sm text-yellow-700">
            This is a simulation for educational purposes. No actual funds will be transferred.
          </p>
        </div>
      </CardContent>
      <CardFooter className="flex flex-col space-y-2">
        <div className="flex justify-between w-full">
          <Button variant="outline" onClick={() => setCurrentStep("education")}>
            Back
          </Button>
          <Button variant="outline" onClick={handleSimulate} disabled={isSimulating}>
            {isSimulating ? "Simulating..." : "Simulate"}
          </Button>
        </div>
        <Button className="w-full" onClick={handleSubmit} disabled={isPending}>
          {isPending ? "Processing..." : "Complete Onboarding"}
        </Button>
      </CardFooter>
    </Card>
  )
}
EOL

cat > components/onboarding/complete-step.tsx << 'EOL'
"use client"

import { useOnboarding } from "@/context/onboarding-context"
import { useWallet } from "@/context/wallet-context"
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { formatAddress } from "@/lib/utils"

export function CompleteStep() {
  const { userData, markComplete } = useOnboarding()
  const { address, ensName } = useWallet()

  const handleComplete = () => {
    markComplete()
    window.location.href = "https://neomi.wtf/dashboard"
  }

  return (
    <Card>
      <CardHeader className="text-center">
        <div className="flex justify-center mb-4">
          <div className="rounded-full bg-green-100 p-3">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              className="h-8 w-8 text-green-600"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
            </svg>
          </div>
        </div>
        <CardTitle>Onboarding Complete!</CardTitle>
        <CardDescription>Congratulations on completing your blockchain journey</CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="p-4 bg-gray-100 rounded-md">
          <h3 className="font-medium mb-2">Your Profile</h3>
          <p className="text-sm">
            <span className="font-medium">Username:</span> {userData.username}
          </p>
          <p className="text-sm">
            <span className="font-medium">Email:</span> {userData.email}
          </p>
          {ensName && (
            <p className="text-sm">
              <span className="font-medium">ENS Name:</span> {ensName}
            </p>
          )}
          <p className="text-sm truncate">
            <span className="font-medium">Wallet:</span> {address ? formatAddress(address) : ""}
          </p>
        </div>

        <div className="space-y-2">
          <p className="text-sm">You've successfully:</p>
          <ul className="list-disc pl-5 space-y-1 text-sm">
            <li>Connected your wallet</li>
            <li>Created your profile</li>
            <li>Completed identity verification</li>
            <li>Learned blockchain basics</li>
            <li>Completed a transaction simulation</li>
          </ul>
        </div>
      </CardContent>
      <CardFooter>
        <Button onClick={handleComplete} className="w-full">
          Go to Dashboard
        </Button>
      </CardFooter>
    </Card>
  )
}
EOL

# Create config files
cat > next.config.js << 'EOL'
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  async redirects() {
    return [
      {
        source: "/",
        destination: "/onboarding",
        permanent: true,
      },
    ]
  },
}

module.exports = nextConfig
EOL

cat > tailwind.config.js << 'EOL'
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "*.{js,ts,jsx,tsx,mdx}",
  ],
  darkMode: ["class"],
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      fontFamily: {
        sans: ["Inter", "sans-serif"],
      },
    },
  },
  plugins: [],
}
EOL

cat > postcss.config.js << 'EOL'
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
EOL

cat > package.json << 'EOL'
{
  "name": "neomi-frontend",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "clsx": "^2.0.0",
    "next": "13.5.4",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "tailwind-merge": "^1.14.0"
  },
  "devDependencies": {
    "@types/node": "^20.8.4",
    "@types/react": "^18.2.28",
    "@types/react-dom": "^18.2.13",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.31",
    "tailwindcss": "^3.3.3",
    "typescript": "^5.2.2"
  }
}
EOL

cat > tsconfig.json << 'EOL'
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
EOL

cat > vercel.json << 'EOL'
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "outputDirectory": ".next",
  "regions": ["iad1"],
  "env": {
    "ONFIDO_API_KEY": "api_live.kQYKpsttlv7.JoPjThs9sSZohBlwhGBX4DvSwbAh5lq3",
    "NODE_ENV": "production"
  }
}
EOL

cat > README.md << 'EOL'
# NEOMI Blockchain Onboarding

A complete blockchain onboarding solution with wallet connection, KYC verification using Onfido, education, and transaction simulation.

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Flisa-neomi%2Fonboardingflow&env=ONFIDO_API_KEY&envDescription=API%20key%20for%20Onfido%20identity%20verification&envLink=https%3A%2F%2Fonfido.com%2Fdevelopers%2F&project-name=neomi-blockchain-onboarding&repository-name=neomi-onboarding)

## Features

- Wallet connection with ENS name support
- KYC verification with Onfido
- Blockchain education modules
- Transaction simulation
- Responsive design

## Deployment

This application is configured to be deployed to neomi.wtf/onboarding using Vercel.

### Environment Variables

The following environment variables are required:

- `ONFIDO_API_KEY`: Your Onfido API key for identity verification

## Development

### Prerequisites

- Node.js 16+ and npm

### Installation

1. Clone the repository
2. Install dependencies:

```bash
npm install
