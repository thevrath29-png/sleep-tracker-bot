import os
import sqlite3
import discord
from discord.ext import commands, tasks
from sleep_plan import PLAN

# Setup SQLite Database
conn = sqlite3.connect("users.db")
cursor = conn.cursor()
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        user_id INTEGER PRIMARY KEY,
        current_day INTEGER DEFAULT 1
    )
""")
conn.commit()

# Bot Setup
intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f"Logged in and ready as {bot.user.name}")
    daily_message_loop.start()

@bot.command()
async def start(ctx):
    """Enrolls the user into the 30-Day Sleep Reset program."""
    user_id = ctx.author.id
    cursor.execute("SELECT current_day FROM users WHERE user_id = ?", (user_id,))
    row = cursor.fetchone()

    if row:
        await ctx.send("You are already enrolled in the 30-Day Sleep Reset!")
    else:
        cursor.execute("INSERT INTO users (user_id, current_day) VALUES (?, 1)", (user_id,))
        conn.commit()
        await ctx.send("Welcome! You've been enrolled. Check your DMs for Day 1!")

        # Send Day 1 DM immediately
        try:
            await ctx.author.send(f"Welcome to **Your 30-Day Sleep Reset**! 🌙\n\n{PLAN[1]}")
        except discord.Forbidden:
            await ctx.send("I couldn't send you a DM! Please enable 'Allow Direct Messages' in your server privacy settings.")

@tasks.loop(hours=24)
async def daily_message_loop():
    """Runs once every 24 hours to deliver the next day's task to each user."""
    cursor.execute("SELECT user_id, current_day FROM users")
    users = cursor.fetchall()

    for user_id, current_day in users:
        next_day = current_day + 1

        if next_day in PLAN:
            try:
                user = await bot.fetch_user(user_id)
                await user.send(f"Here is your daily task:\n\n{PLAN[next_day]}")
                
                # Advance user progress by 1 day
                cursor.execute("UPDATE users SET current_day = ? WHERE user_id = ?", (next_day, user_id))
                conn.commit()
            except Exception as e:
                print(f"Failed to deliver message to {user_id}: {e}")
        else:
            # Reached day 30 completion
            try:
                user = await bot.fetch_user(user_id)
                await user.send("🎉 **Congratulations!** You've finished the 30-Day Sleep Reset. Remember: if life gets chaotic, protect your top habit first!")
            except Exception:
                pass
            cursor.execute("DELETE FROM users WHERE user_id = ?", (user_id,))
            conn.commit()

@daily_message_loop.before_loop
async def before_daily_loop():
    await bot.wait_until_ready()

# Pass DISCORD_TOKEN via your environment variables or host service
bot.run(os.getenv("DISCORD_TOKEN"))