// Menggunakan node-fetch v2 (CommonJS)
const fetch = require('node-fetch');

// Ini adalah Serverless Function
// Vercel akan otomatis menangani CORS, jadi kita bisa sederhanakan
export default async function handler(req, res) {

    // Hanya izinkan metode POST
    if (req.method !== 'POST') {
        res.setHeader('Allow', ['POST']);
        return res.status(405).end('Method Not Allowed');
    }

    try {
        // 1. Ambil Kunci API dari Environment Variables Vercel
        const GOOGLE_API_KEY = process.env.GOOGLE_API_KEY;
        if (!GOOGLE_API_KEY) {
            console.error("FATAL ERROR: GOOGLE_API_KEY tidak ditemukan di Vercel Environment Variables.");
            return res.status(500).json({ error: "Server tidak terkonfigurasi dengan benar." });
        }
        
        const GOOGLE_API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${GOOGLE_API_KEY}`;

        // 2. Ambil data prompt dari body permintaan
        const { systemPrompt, userPrompt } = req.body;

        if (!userPrompt) {
            return res.status(400).json({ error: "userPrompt tidak ditemukan" });
        }

        // 3. Siapkan payload untuk Google API
        const googlePayload = {
            contents: [{ 
                parts: [{ text: `${systemPrompt}\n\n${userPrompt}` }] 
            }]
        };

        // 4. Panggil Google API dari server (Aman)
        const apiResponse = await fetch(GOOGLE_API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(googlePayload)
        });

        const responseData = await apiResponse.json(); // Baca JSON sekali

        if (!apiResponse.ok) {
            console.error("Error dari Google API:", responseData);
            throw new Error(`Google API error: ${apiResponse.status} ${responseData.error?.message || apiResponse.statusText}`);
        }

        // 5. Ekstrak teks balasan
        const text = responseData.candidates?.[0]?.content?.parts?.[0]?.text;

        if (!text) {
            console.error("Respon Google tidak valid:", responseData);
            throw new Error("Tidak ada teks balasan yang valid dari Google API.");
        }

        // 6. Kirim balasan (hanya teks) kembali ke frontend
        res.status(200).json({ text: text.trim() });

    } catch (error) {
        console.error("Error di /api/generate:", error.message);
        res.status(500).json({ error: "Terjadi kesalahan di server", details: error.message });
    }
}
