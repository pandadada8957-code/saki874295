import asyncio
import os
import random
import discord
from openai import OpenAI

# 環境変数の読み込み
TOKEN_A = os.getenv("BOT_TOKEN_A")
TOKEN_B = os.getenv("BOT_TOKEN_B")
OPENAI_KEY = os.getenv("OPENAI_API_KEY")

CHANNEL_ID = int(os.getenv("CHANNEL_ID", "0"))  # 会話させたいチャンネルID
MAX_TURNS = 6  # 1回の会話の最大往復数
turn_count = 0

openai_client = OpenAI(api_key=OPENAI_KEY)
intents = discord.Intents.default()
intents.message_content = True

client_a = discord.Client(intents=intents)
client_b = discord.Client(intents=intents)

SYSTEM_A = "あなたは明るくフレンドリーなAI「ルナ」です。短めの口語で返答してください。"
SYSTEM_B = (
    "あなたは少しクールで知識豊富なAI「レイ」です。短めの口語で返答してください。"
)


async def get_ai_response(system_prompt, user_text):
  response = openai_client.chat.completions.create(
      model="gpt-4o-mini",
      messages=[
          {"role": "system", "content": system_prompt},
          {"role": "user", "content": user_text},
      ],
      max_tokens=150,
  )
  return response.choices[0].message.content


@client_a.event
async def on_message(message):
  global turn_count
  if message.channel.id != CHANNEL_ID:
    return
  if message.author == client_b.user:
    if turn_count >= MAX_TURNS:
      turn_count = 0
      return
    turn_count += 1
    await asyncio.sleep(3)
    async with message.channel.typing():
      reply = await get_ai_response(SYSTEM_A, message.content)
      await message.channel.send(reply)


@client_b.event
async def on_message(message):
  global turn_count
  if message.channel.id != CHANNEL_ID:
    return
  if message.author == client_a.user:
    if turn_count >= MAX_TURNS:
      return
    turn_count += 1
    await asyncio.sleep(3)
    async with message.channel.typing():
      reply = await get_ai_response(SYSTEM_B, message.content)
      await message.channel.send(reply)


async def auto_start():
  await client_a.wait_until_ready()
  channel = client_a.get_channel(CHANNEL_ID)
  while not client_a.is_closed():
    await asyncio.sleep(3600)  # 1時間ごとに会話を開始
    global turn_count
    turn_count = 0
    topics = ["最近の流行り", "おすすめのゲーム", "美味しい食べ物"]
    topic = random.choice(topics)
    async with channel.typing():
      reply = await get_ai_response(
          SYSTEM_A, f"次のトピックで相手に話しかけて: {topic}"
      )
      await channel.send(reply)


async def main():
  client_a.loop.create_task(auto_start())
  await asyncio.gather(
      client_a.start(TOKEN_A),
      client_b.start(TOKEN_B),
  )


if __name__ == "__main__":
  asyncio.run(main())
