"use client"

import type React from "react"

import { useState, useEffect, useRef } from "react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Label } from "@/components/ui/label"
import { Alert, AlertDescription } from "@/components/ui/alert"
import { RefreshCw, CheckCircle2, XCircle } from "lucide-react"
import { generateCaptcha, verifyCaptcha } from "@/lib/captcha-actions"

// CAPTCHA Display Component
function CaptchaDisplay({ text }: { text: string }) {
  const canvasRef = useRef<HTMLCanvasElement>(null)

  useEffect(() => {
    const canvas = canvasRef.current
    if (!canvas) return

    const ctx = canvas.getContext("2d")
    if (!ctx) return

    // Clear canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height)

    // Set background
    ctx.fillStyle = "white"
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    // Add noise (dots)
    for (let i = 0; i < 100; i++) {
      ctx.fillStyle = `rgba(${Math.random() * 200}, ${Math.random() * 200}, ${Math.random() * 200}, 0.3)`
      ctx.beginPath()
      ctx.arc(Math.random() * canvas.width, Math.random() * canvas.height, Math.random() * 2, 0, Math.PI * 2)
      ctx.fill()
    }

    // Add lines for noise
    for (let i = 0; i < 4; i++) {
      ctx.strokeStyle = `rgba(${Math.random() * 200}, ${Math.random() * 200}, ${Math.random() * 200}, 0.5)`
      ctx.lineWidth = 1
      ctx.beginPath()
      ctx.moveTo(Math.random() * canvas.width, Math.random() * canvas.height)
      ctx.lineTo(Math.random() * canvas.width, Math.random() * canvas.height)
      ctx.stroke()
    }

    // Draw text
    const fontSize = 24
    ctx.font = `${fontSize}px 'Courier New', monospace`
    ctx.textBaseline = "middle"

    // Draw each character with slight variations
    const chars = text.split("")
    const charWidth = canvas.width / (chars.length + 1)

    chars.forEach((char, i) => {
      const x = charWidth * (i + 0.5)
      const y = canvas.height / 2 + (Math.random() * 10 - 5)
      const rotation = Math.random() * 0.4 - 0.2

      ctx.save()
      ctx.translate(x, y)
      ctx.rotate(rotation)
      ctx.fillStyle = `rgb(${Math.floor(Math.random() * 80)}, ${Math.floor(Math.random() * 80)}, ${Math.floor(Math.random() * 80)})`
      ctx.fillText(char, -8, 0)
      ctx.restore()
    })
  }, [text])

  return <canvas ref={canvasRef} width={200} height={80} className="w-full h-auto" aria-label="CAPTCHA image" />
}

// Main Page Component with CAPTCHA Form
export default function CaptchaPage() {
  const [captchaText, setCaptchaText] = useState("")
  const [userInput, setUserInput] = useState("")
  const [isVerified, setIsVerified] = useState<boolean | null>(null)
  const [isLoading, setIsLoading] = useState(false)

  const refreshCaptcha = async () => {
    setIsLoading(true)
    try {
      const newCaptcha = await generateCaptcha()
      setCaptchaText(newCaptcha)
      setUserInput("")
      setIsVerified(null)
    } catch (error) {
      console.error("Failed to generate CAPTCHA:", error)
    } finally {
      setIsLoading(false)
    }
  }

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsLoading(true)

    try {
      const result = await verifyCaptcha(captchaText, userInput)
      setIsVerified(result)

      if (result) {
        // If verified, you could proceed with form submission or other actions
        setTimeout(() => {
          refreshCaptcha()
        }, 2000)
      }
    } catch (error) {
      console.error("Verification failed:", error)
      setIsVerified(false)
    } finally {
      setIsLoading(false)
    }
  }

  useEffect(() => {
    refreshCaptcha()
  }, [])

  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-4 bg-gray-50">
      <div className="w-full max-w-md space-y-8">
        <div className="text-center">
          <h1 className="text-3xl font-bold">CAPTCHA Demo</h1>
          <p className="mt-2 text-gray-600">A simple implementation of CAPTCHA verification</p>
        </div>

        <Card>
          <CardHeader>
            <CardTitle>Verify you're human</CardTitle>
            <CardDescription>Enter the characters you see in the image below</CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleSubmit} className="space-y-6">
              <div className="space-y-2">
                <div className="flex justify-between items-center">
                  <Label htmlFor="captcha">CAPTCHA</Label>
                  <Button type="button" variant="ghost" size="sm" onClick={refreshCaptcha} disabled={isLoading}>
                    <RefreshCw className={`h-4 w-4 ${isLoading ? "animate-spin" : ""}`} />
                    <span className="sr-only">Refresh CAPTCHA</span>
                  </Button>
                </div>

                <div className="border rounded-md p-4 bg-white">
                  {captchaText ? (
                    <CaptchaDisplay text={captchaText} />
                  ) : (
                    <div className="h-16 flex items-center justify-center">
                      <p className="text-gray-400">Loading CAPTCHA...</p>
                    </div>
                  )}
                </div>
              </div>

              <div className="space-y-2">
                <Label htmlFor="captcha-input">Enter the text above</Label>
                <Input
                  id="captcha-input"
                  value={userInput}
                  onChange={(e) => setUserInput(e.target.value)}
                  placeholder="Enter CAPTCHA text"
                  disabled={isLoading}
                  className="w-full"
                />
              </div>

              {isVerified !== null && (
                <Alert
                  className={
                    isVerified ? "bg-green-50 text-green-800 border-green-200" : "bg-red-50 text-red-800 border-red-200"
                  }
                >
                  <div className="flex items-center gap-2">
                    {isVerified ? (
                      <CheckCircle2 className="h-4 w-4 text-green-600" />
                    ) : (
                      <XCircle className="h-4 w-4 text-red-600" />
                    )}
                    <AlertDescription>
                      {isVerified ? "CAPTCHA verified successfully!" : "Incorrect CAPTCHA. Please try again."}
                    </AlertDescription>
                  </div>
                </Alert>
              )}

              <Button type="submit" className="w-full" disabled={!captchaText || !userInput || isLoading}>
                Verify
              </Button>
            </form>
          </CardContent>
        </Card>
      </div>
    </main>
  )
}
