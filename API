import { google } from "googleapis"
import { type NextRequest, NextResponse } from "next/server"

// Define CORS headers
const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type, Authorization",
}

// Handle OPTIONS requests (preflight)
export async function OPTIONS() {
  return new NextResponse(null, {
    status: 204,
    headers: corsHeaders,
  })
}

export async function POST(request: NextRequest) {
  try {
    // Parse request body
    const body = await request.json()
    const { orgId, licenseKey } = body

    // Validate required fields
    if (!orgId || !licenseKey) {
      return new NextResponse(JSON.stringify({ error: "Missing orgId or licenseKey" }), {
        status: 400,
        headers: {
          "Content-Type": "application/json",
          ...corsHeaders,
        },
      })
    }

    // Get environment variables
    const sheetId = process.env.GOOGLE_SHEET_ID
    const serviceAccountJson = process.env.GOOGLE_SERVICE_ACCOUNT

    if (!sheetId || !serviceAccountJson) {
      return new NextResponse(JSON.stringify({ error: "Missing required environment variables" }), {
        status: 500,
        headers: {
          "Content-Type": "application/json",
          ...corsHeaders,
        },
      })
    }

    // Parse service account JSON
    const serviceAccount = JSON.parse(serviceAccountJson)

    // Set up Google Sheets authentication
    const auth = new google.auth.GoogleAuth({
      credentials: serviceAccount,
      scopes: ["https://www.googleapis.com/auth/spreadsheets.readonly"],
    })

    const sheets = google.sheets({ version: "v4", auth })

    // Fetch data from Google Sheet
    const response = await sheets.spreadsheets.values.get({
      spreadsheetId: sheetId,
      range: "Sheet1!A2:D",
    })

    const rows = response.data.values

    if (!rows || rows.length === 0) {
      return new NextResponse(JSON.stringify({ error: "No license records found." }), {
        status: 404,
        headers: {
          "Content-Type": "application/json",
          ...corsHeaders,
        },
      })
    }

    // Find matching license
    const match = rows.find((row) => row[0] === orgId && row[1] === licenseKey)

    if (!match) {
      return new NextResponse(
        JSON.stringify({
          valid: false,
          reason: "Invalid license",
        }),
        {
          status: 403,
          headers: {
            "Content-Type": "application/json",
            ...corsHeaders,
          },
        },
      )
    }

    // Extract license details
    const workerLimit = Number.parseInt(match[2], 10)
    const expires = match[3]

    return new NextResponse(
      JSON.stringify({
        valid: true,
        workerLimit,
        expires,
      }),
      {
        status: 200,
        headers: {
          "Content-Type": "application/json",
          ...corsHeaders,
        },
      },
    )
  } catch (error) {
    console.error("License validation error:", error)
    return new NextResponse(
      JSON.stringify({
        error: error instanceof Error ? error.message : "Unknown error",
      }),
      {
        status: 500,
        headers: {
          "Content-Type": "application/json",
          ...corsHeaders,
        },
      },
    )
  }
}
