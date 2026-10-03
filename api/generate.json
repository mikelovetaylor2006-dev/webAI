import { GoogleGenAI } from '@google/genai';
import Replicate from 'replicate';

export default async function handler(req, res) {
  // Thêm Header cho phép gọi API từ mọi nguồn
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ success: false, error: 'Method not allowed' });
  }

  const { script } = req.body || {};
  if (!script || script.trim() === '') {
    return res.status(400).json({ success: false, error: 'Vui lòng nhập kịch bản đầu vào' });
  }

  try {
    // 1. Phân tích kịch bản bằng Gemini API
    const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });
    const promptSystem = `Bạn là một đạo diễn phim AI. Hãy chuyển đổi kịch bản sau thành 1 câu Visual Prompt bằng tiếng Anh chi tiết (góc quay, ánh sáng, 8k, điện ảnh) để đưa vào AI Text-to-Video: "${script}"`;
    
    const geminiResponse = await ai.models.generateContent({
      model: 'gemini-2.5-flash',
      contents: promptSystem,
    });
    
    const generatedPrompt = geminiResponse.text ? geminiResponse.text.trim() : script;

    // 2. Gửi lệnh sang Replicate API
    const replicate = new Replicate({
      auth: process.env.REPLICATE_API_TOKEN,
    });

    const prediction = await replicate.predictions.create({
      version: "9f7709c00b01e7e40822363152594a11c1e5a519bfb4b5768565b4c4897e93a6",
      input: {
        prompt: generatedPrompt,
        num_frames: 81,
        frames_per_second: 16
      }
    });

    return res.status(200).json({ 
      success: true, 
      predictionId: prediction.id,
      promptUsed: generatedPrompt
    });

  } catch (error) {
    console.error('API Error:', error);
    return res.status(500).json({ 
      success: false, 
      error: error.message || 'Lỗi xử lý Serverless' 
    });
  }
}
