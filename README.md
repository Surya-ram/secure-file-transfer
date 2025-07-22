# Secure file transfer system
## Name : Surya R
## Register no : 212222040167
# Aim:
A secure file-sharing web application that allows users to upload, encrypt, and share files via a one-time download link. Files are encrypted in the browser and decrypted only on the recipient's side.
# Methodology:
The project follows a client-server architecture with strict security principles, implemented using Bolt.new (a React + API framework). The development approach is modular, secure-by-design, and adheres to the principles of end-to-end encryption and minimal server trust.
# Steps for file transfer workflow :
1. User selects a file to upload.
2. File is encrypted in the browser using AES-GCM.
3. Encrypted data and IV are sent to the server.
4. Server stores encrypted data in-memory temporarily.
5. A one-time link is generated containing a decryption key.
6. Recipient uses the link to decrypt and download the file.
# Technologies Used :
- Bolt.new (Full-stack platform)
- React + Tailwind CSS (Frontend)
- TypeScript (Backend API)
- Web Crypto API (Encryption)
# Code Archietecture :
- upload.tsx: Encrypts and uploads the file
- download.tsx: Downloads and decrypts the file
- api/upload.ts: Temporarily stores encrypted data
## Code :
#  upload.tsx – Secure File Upload with AES Encryption
```
'use client'
import { useState } from 'react'

export default function UploadPage() {
  const [file, setFile] = useState<File | null>(null)
  const [link, setLink] = useState("")

  async function handleUpload() {
    if (!file) return
    const key = crypto.getRandomValues(new Uint8Array(32))
    const encoded = new TextEncoder().encode(await file.text())
    const iv = crypto.getRandomValues(new Uint8Array(12))

    const encrypted = await crypto.subtle.encrypt(
      { name: "AES-GCM", iv },
      await crypto.subtle.importKey("raw", key, "AES-GCM", false, ["encrypt"]),
      encoded
    )

    const res = await fetch("/api/upload", {
      method: "POST",
      body: JSON.stringify({
        data: Array.from(new Uint8Array(encrypted)),
        iv: Array.from(iv),
      }),
    })

    const { id } = await res.json()
    const encodedKey = btoa(String.fromCharCode(...key))
    setLink(`${location.origin}/download/${id}#${encodedKey}`)
  }

  return (
    <div className="p-4">
      <h1 className="text-xl font-bold">Secure File Upload</h1>
      <input type="file" onChange={(e) => setFile(e.target.files?.[0] || null)} />
      <button className="mt-2 bg-blue-600 text-white p-2 rounded" onClick={handleUpload}>Encrypt & Upload</button>
      {link && <p className="mt-4">Share this link: <a href={link} className="text-blue-500">{link}</a></p>}
    </div>
  )
}
```
# api/upload.ts – Store Encrypted File in Memory
```
let storage: Record<string, { data: number[], iv: number[] }> = {}

export async function POST(req: Request) {
  const body = await req.json()
  const id = crypto.randomUUID()
  storage[id] = { data: body.data, iv: body.iv }
  return Response.json({ id })
}

export { storage }
```
# download.tsx – Decrypt File in Browser
```
'use client'
import { useEffect, useState } from 'react'
import { useParams } from 'next/navigation'

export default function DownloadPage() {
  const { id } = useParams()
  const [fileText, setFileText] = useState("")

  useEffect(() => {
    const keyStr = location.hash.substring(1)
    if (!keyStr) return

    const keyBytes = Uint8Array.from(atob(keyStr), c => c.charCodeAt(0))
    fetch(`/api/download?id=${id}`)
      .then(res => res.json())
      .then(async ({ data, iv }) => {
        const decrypted = await crypto.subtle.decrypt(
          { name: "AES-GCM", iv: new Uint8Array(iv) },
          await crypto.subtle.importKey("raw", keyBytes, "AES-GCM", false, ["decrypt"]),
          new Uint8Array(data)
        )
        setFileText(new TextDecoder().decode(decrypted))
      })
  }, [id])

  return (
    <div className="p-4">
      <h1 className="text-xl font-bold">Decrypted File:</h1>
      <pre className="bg-gray-100 p-2 mt-2">{fileText || "Decrypting..."}</pre>
    </div>
  )
}
```
# Output
<img width="1650" height="839" alt="image" src="https://github.com/user-attachments/assets/1a78060b-f959-45e4-ae0e-d6557e2b62e7" />
<img width="1653" height="849" alt="image" src="https://github.com/user-attachments/assets/0aa77361-9ad2-4bb8-a819-e9610e388b00" />

# Live demo : https://bolt.new/~/sb1-suxhby8y
# Features :
1. Drag & drop file upload with security validation
2.Real-time encryption status and security metrics
3.File integrity verification with hash generation
4.Secure transfer simulation with progress tracking
5.Authentication simulation with security levels
6.Security best practices and protocol information
7.Threat detection and security warnings
# Result :
This project effectively demonstrates core cybersecurity principles such as encryption, secure key exchange, and temporary data handling using a modern web framework. It is suitable for secure peer-to-peer file sharing in bothpersonal and professional contexts.




