import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const SYSTEM_PROMPT = `You are Qlviro Pulse — a creator screener for service-business operators.

Your job: given a creator's social handle, use web_search to research them across Instagram, TikTok, X/Twitter, YouTube, LinkedIn — whichever platforms surface. Then score them on 7 filters (1–5 each).

THE 7 FILTERS:
1. engagement — likes + comments ÷ followers. 1 = dead account, 5 = red-hot ratio (>5%).
2. pain_signal — does the comment-section language show buying intent or problem awareness? 1 = pure fluff/emojis, 5 = audience openly describing pain the creator could solve.
3. pay_capacity — audience location + niche economy. 1 = broke audience, 5 = premium spend (US/UK/AU professionals, B2B operators, etc).
4. niche_clarity — does the bio + recent posts tell ONE coherent story? 1 = scattered/lifestyle dump, 5 = laser-focused on a single ICP.
5. active_monetiser — visible Linktree, paid product, course, community, lead magnet? 1 = nothing, 5 = full funnel with multiple offers.
6. posting_cadence — at least 2x/week? 1 = ghost account, 5 = daily / multi-platform discipline.
7. qlviro_fit — does the audience overlap with service-business operators (the Qlviro ICP)? 1 = zero overlap, 5 = perfect match.

VERDICT BANDS (sum of all 7 scores, out of 35):
- PASS: 28+
- MAYBE: 22–27
- KILL: <22

OUTPUT FORMAT — STRICT:
Return ONLY a single valid JSON object. No prose, no markdown code fences, no leading/trailing text. Exact schema:

{
  "handle": "<the handle you screened, normalised with leading @>",
  "scores": {
    "engagement":        { "score": <int 1-5>, "reasoning": "<max 2 lines>" },
    "pain_signal":       { "score": <int 1-5>, "reasoning": "<max 2 lines>" },
    "pay_capacity":      { "score": <int 1-5>, "reasoning": "<max 2 lines>" },
    "niche_clarity":     { "score": <int 1-5>, "reasoning": "<max 2 lines>" },
    "active_monetiser":  { "score": <int 1-5>, "reasoning": "<max 2 lines>" },
    "posting_cadence":   { "score": <int 1-5>, "reasoning": "<max 2 lines>" },
    "qlviro_fit":        { "score": <int 1-5>, "reasoning": "<max 2 lines>" }
  },
  "total": <int 0-35, must equal sum of all 7 scores>,
  "verdict": "PASS" | "MAYBE" | "KILL"
}

If research fails or the creator can't be found, still return valid JSON with conservative scores (1s) and a reasoning line of "Insufficient public data found" per filter. Never break schema.`;

export default async function handler(req, res) {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") return res.status(200).end();
  if (req.method !== "POST") {
    return res.status(405).json({ error: "Method not allowed. Use POST." });
  }

  let body = req.body;
  if (typeof body === "string") {
    try {
      body = JSON.parse(body);
    } catch {
      return res.status(400).json({ error: "Invalid JSON body" });
    }
  }

  const rawHandle = body?.handle;
  if (!rawHandle || typeof rawHandle !== "string" || rawHandle.trim().length === 0) {
    return res.status(400).json({ error: "Missing 'handle' in request body" });
  }

  const handle = rawHandle.trim();

  if (!process.env.ANTHROPIC_API_KEY) {
    return res.status(500).json({ error: "Server missing ANTHROPIC_API_KEY" });
  }

  try {
    const response = await client.messages.create({
      model: "claude-haiku-4-5-20251001",
      max_tokens: 2000,
      system: SYSTEM_PROMPT,
      tools: [
        {
          type: "web_search_20250305",
          name: "web_search",
          max_uses: 8,
        },
      ],
      messages: [
        {
          role: "user",
          content: `Research and score this creator: ${handle}

Search for their primary platform first (Instagram if it looks like an @handle, otherwise infer). Pull: follower count, recent post engagement, bio, comment patterns, visible monetisation, posting frequency, audience signals (location, profession).

Return the JSON scorecard. JSON only — no prose.`,
        },
      ],
    });

    const finalText = response.content
      .filter((b) => b.type === "text")
      .map((b) => b.text)
      .join("\n")
      .trim();

    if (!finalText) {
      return res.status(502).json({
        error: "Model returned no text output",
        stop_reason: response.stop_reason,
      });
    }

    let cleaned = finalText
      .replace(/^```(?:json)?\s*/i, "")
      .replace(/```\s*$/i, "")
      .trim();

    if (!cleaned.startsWith("{")) {
      const match = cleaned.match(/\{[\s\S]*\}/);
      if (match) cleaned = match[0];
    }

    let parsed;
    try {
      parsed = JSON.parse(cleaned);
    } catch (parseErr) {
      return res.status(502).json({
        error: "Failed to parse model output as JSON",
        raw: finalText,
      });
    }

    const filterKeys = [
      "engagement",
      "pain_signal",
      "pay_capacity",
      "niche_clarity",
      "active_monetiser",
      "posting_cadence",
      "qlviro_fit",
    ];

    let total = 0;
    for (const k of filterKeys) {
      const s = parsed?.scores?.[k]?.score;
      if (typeof s === "number") total += s;
    }
    parsed.total = total;
    parsed.verdict = total >= 28 ? "PASS" : total >= 22 ? "MAYBE" : "KILL";

    return res.status(200).json(parsed);
  } catch (err) {
    console.error("[pulse] error:", err);
    return res.status(500).json({
      error: err?.message || "Unknown server error",
    });
  }
}

export const maxDuration = 60;
