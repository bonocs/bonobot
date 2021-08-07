import sys
import discord
import DiscordUtils
from discord.ext import commands, tasks
import asyncio
from datetime import datetime, timezone
import pytz
import logging


EST = pytz.timezone('US/Eastern')
bot = commands.Bot(command_prefix = '.')

logger = logging.getLogger(__name__)

def read_token():
    with open("token.txt", "r") as f:
        lines = f.readlines()
        return lines[0].strip()


token = read_token()
intents = discord.Intents.default()
intents.members = True
music = DiscordUtils.Music()
client = discord.Client(intents=intents)


class DurationConverter(commands.Converter):
    async def convert(self, ctx, argument):
        amount = argument[:-1]
        unit = argument[-1]

        if amount.isdigit() and unit in ['s', 'm']:
            return (int(amount), unit)

        raise commands.BadArgument(message='Not a valid duration')


@bot.event
async def on_ready():
    await bot.change_presence(activity=discord.Activity(type=discord.ActivityType.watching, name=("anthony grow hair")))
    logger.warning(f"BotLaunched::")

@bot.command()
async def s(ctx):
    player = music.get_player(guild_id=ctx.guild.id)
    song = await player.toggle_song_loop()
    song.is_looping = False
    print(song.is_looping)
    print(player.queue)
    song2 = await player.remove_from_queue(int(0))
    await ctx.send(f'{song2.name} was skipped nooooooob')

@bot.command()
async def r(ctx, index):
    player = music.get_player(guild_id=ctx.guild.id)
    song = await player.remove_from_queue(int(index))
    await ctx.send(f'{song.name} was removed from the queue lmao bald')


@bot.command()
async def p(ctx, *, url):
    player = music.get_player(guild_id=ctx.guild)
    if ctx.voice_client is None:
        await ctx.author.voice.channel.connect()
    if not player:
        player = music.create_player(ctx, ffmpeg_error_betterfix=True)
    if not ctx.voice_client.is_playing():
        await player.queue(url, search=True)
        try:
            song = await player.play()
        except:
            logger.error("Unexpected error:", sys.exc_info()[0])
            raise
        await ctx.send(f"hey baldthony '{song.name}' is now playing lmaooooooo baldthony")
    else:
        song = await player.queue(url, search=True)
        await ctx.send(f"'{song.name}' has been added to the queue lmao nooby fortnite player")


@bot.command()
async def l(ctx):
    player = music.get_player(guild_id=ctx.guild.id)
    song = await player.toggle_song_loop()
    if song.is_looping:
        return await ctx.send("haha song is looping noobs")
    else:
        return await ctx.send("song isnt looping anymore bald")


@bot.command()
async def q(ctx):
    player = music.get_player(guild_id=ctx.guild.id)
    await ctx.send(f"{', '.join([song.name for song in player.current_queue()])}")

@bot.command()
async def np(ctx):
    player = music.get_player(guild_id=ctx.guild.id)
    song = player.now_playing()
    await ctx.send(song.name)

@bot.command()
async def bono(ctx):
    embed = discord.Embed(
        title="Command Syntax",
        description="**.mute** \n @user optional reason \n *Example: .mute @bono 4m because I can!* \n \n **.unmute** \n @user \n *Example: .unmute @bono* \n \n **.tempmute** \n @user time(s, m, h) optional reason \n *Example: .tempmute @bono 46s I felt like it*"
    )
    await ctx.send(embed=embed)

@bot.event
@commands.has_permissions(manage_messages=True)
async def on_command_error(ctx, error):
    if isinstance(error, commands.MemberNotFound):
        embed = discord.Embed(
            title="discord.ext.commands.errors.MemberNotFound",
            description="Either the member doesn't exist or you are using invalid syntax.",
            timestamp=datetime.now().astimezone(EST)
        )
        await ctx.send(embed=embed)

@bot.command()
@commands.has_permissions(manage_messages=True)
async def mute(ctx, member: discord.Member, *, reason=None):

    guild = ctx.guild
    mutedRole = discord.utils.get(guild.roles, name="Muted")

    if not mutedRole:
        mutedRole = await guild.create_tole(name="Muted")

        for channel in guild.channels:
            await channel.set_permissions(mutedRole, speak=False, send_messages=False, read_message_history=True, read_messages=True)

    await member.add_roles(mutedRole, reason=None)
    embed = discord.Embed(
        title="Member Muted",
        description=(f"Member {member.mention} was muted for {reason}."),
        timestamp=datetime.now().astimezone(EST)
    )
    await ctx.send(embed=embed)


@bot.command()
@commands.has_permissions(manage_messages=True)
async def unmute(ctx, member: discord.Member):
    guild = ctx.guild
    mutedRole = discord.utils.get(guild.roles, name="Muted")
    await member.remove_roles(mutedRole)
    embed = discord.Embed(
        title="Member Unmuted",
        description=(f"Member {member.mention} has been unmuted.")
    )
    await ctx.send(embed=embed)

@bot.command(description="Mutes the specified user.")
@commands.has_permissions(manage_messages=True)
async def tempmute(ctx, member: discord.Member, timed: DurationConverter, *, reason=None):
    guild = ctx.guild
    mutedRole = discord.utils.get(guild.roles, name="Muted")

    if not mutedRole:
        mutedRole = await guild.create_role(name="Muted")

        for channel in guild.channels:
            await channel.set_permissions(mutedRole, speak=False, send_messages=False, read_message_history=True, read_messages=True)

    multiplier = {'s': 1, 'm': 60, 'h': 3600}
    amount, unit = timed

    await member.add_roles(mutedRole, reason=None)
    embed = discord.Embed(
        title="Member Muted",
        description=(f"Member {member.mention} was muted for {amount}{unit}, reason {reason}."),
        timestamp=datetime.now().astimezone(EST)
    )
    await ctx.send(embed=embed)

    mute_time = amount * multiplier[unit]
    print(mute_time)
    await asyncio.sleep(amount * multiplier[unit])
    print("User Unmuted")

    await member.remove_roles(mutedRole)
    embed = discord.Embed(
        title="Member Unmuted",
        description=(f"Member {member.mention} has been unmuted, mute reason '{reason}.'"),
        timestamp=datetime.now().astimezone(EST)
    )
    await ctx.send(embed=embed)

@bot.command(pass_context=True)
async def verify(ctx):
    embed = discord.Embed(
        title="Hello!",
        description="Click the reaction below to access the rest of the server.",
    )
    msg = await ctx.send(embed=embed)
    await msg.add_reaction('🤔')


@bot.command(pass_context=True)
async def reaction(ctx):
    embed = discord.Embed(
        title="Hello!",
        description="Click on the reactions for a role, if you remove your reaction the role gets removed. \n 🔺 - Light Red \n 🟥 - Red \n 🔴 - Dark Red \n 🔶 - Light Orange \n 🟧 - Orange \n 🟠 - Dark Orange \n 💛 - Light Yellow \n 🟨 - Yellow \n 🟡 - Dark Yellow \n 🍏 - Light Green \n 🟩 - Green \n 🟢 - Dark Green \n 🔷 - Light Blue \n 🟦 - Blue \n 🔵 - Dark Blue \n 💜 - Light Purple \n 🟪 - Purple \n 🟣 - Dark Purple \n 🦑 - Pink \n 🐌 - Dark Pink",
    )
    msg = await ctx.send(embed=embed)
    await msg.add_reaction('🔺')
    await msg.add_reaction('🟥')
    await msg.add_reaction('🔴')
    await msg.add_reaction('🔶')
    await msg.add_reaction('🟧')
    await msg.add_reaction('🟠')
    await msg.add_reaction('💛')
    await msg.add_reaction('🟨')
    await msg.add_reaction('🟡')
    await msg.add_reaction('🍏')
    await msg.add_reaction('🟩')
    await msg.add_reaction('🟢')
    await msg.add_reaction('🔷')
    await msg.add_reaction('🟦')
    await msg.add_reaction('🔵')
    await msg.add_reaction('💜')
    await msg.add_reaction('🟪')
    await msg.add_reaction('🟣')
    await msg.add_reaction('🦑')
    await msg.add_reaction('🐌')


@bot.command(pass_context=True)
async def setup(ctx):
    global channelid
    global channelid2

    channel2 = discord.utils.get(ctx.guild.channels, name="reaction-roles")
    channel = discord.utils.get(ctx.guild.channels, name="verify")

    if not channel:
        guild = ctx.guild
        await guild.create_text_channel('verify')
        await ctx.send("Channel 'verify' succesfully created, run this command again.")
    if not channel2:
        guild = ctx.guild
        await guild.create_text_channel('reaction-roles')
        await ctx.send("Channel 'reaction-roles' succesfully created, run this command again.")

    channelid2 = channel2.id
    channelid = channel.id

    await ctx.send("Searching for a verify channel...")
    await ctx.send(f"Verify channel found, ID: {channelid}. Run the .verify command in the #verify channel to create a verify prompt.")
    await ctx.send("Searching for a reaction-roles channel...")
    await ctx.send(f"Reaction-Roles channel found, ID: {channelid2}. Run the .reaction command in the #reaction-roles channel to create a reaction-role prompt.")

@bot.event
async def on_raw_reaction_add(payload):
    global channelid
    global channelid2

    print(payload.channel_id)
    if payload.channel_id == channelid:
        print("verify channel")
        await on_raw_reaction_add_verify(payload)
    elif payload.channel_id == channelid2:
        await on_raw_reaction_add_reaction(payload)
    elif payload.channel_id == channelid2:
        await on_raw_reaction_remove(payload)
    else:
        print("this message doesnt exist")

# this is the verify command

async def on_raw_reaction_add_verify(payload):
    print(payload.emoji)
    member = payload.member
    guild = member.guild
    emoji = payload.emoji.name
    if emoji == '🤔':
        role = discord.utils.get(guild.roles, name="Members")
        await member.add_roles(role)


# this is the role reaction command, currently transitioning this to cogs

async def on_raw_reaction_add_reaction(payload):
    print(payload.emoji)
    member = payload.member
    guild = member.guild

    emoji = payload.emoji.name
    if emoji == '🔺':
        role = discord.utils.get(guild.roles, name="Light Red")
        await member.add_roles(role)
    if emoji == '🟥':
        role = discord.utils.get(guild.roles, name="Red")
        await member.add_roles(role)
    if emoji == '🔴':
        role = discord.utils.get(guild.roles, name="Dark Red")
        await member.add_roles(role)
    if emoji == '🟦':
        role = discord.utils.get(guild.roles, name="Blue")
        await member.add_roles(role)
    if emoji == '🔵':
        role = discord.utils.get(guild.roles, name="Dark Blue")
        await member.add_roles(role)
    if emoji == '🔷':
        role = discord.utils.get(guild.roles, name="Light Blue")
        await member.add_roles(role)
    if emoji == '🟧':
        role = discord.utils.get(guild.roles, name="Orange")
        await member.add_roles(role)
    if emoji == '🔶':
        role = discord.utils.get(guild.roles, name="Light Orange")
        await member.add_roles(role)
    if emoji == '🟠':
        role = discord.utils.get(guild.roles, name="Dark Orange")
        await member.add_roles(role)
    if emoji == '💛':
        role = discord.utils.get(guild.roles, name="Light Yellow")
        await member.add_roles(role)
    if emoji == '🟨':
        role = discord.utils.get(guild.roles, name="Yellow")
        await member.add_roles(role)
    if emoji == '🟡':
        role = discord.utils.get(guild.roles, name="Dark Yellow")
        await member.add_roles(role)
    if emoji == '🟩':
        role = discord.utils.get(guild.roles, name="Green")
        await member.add_roles(role)
    if emoji == '🍏':
        role = discord.utils.get(guild.roles, name="Light Green")
        await member.add_roles(role)
    if emoji == '🟢':
        role = discord.utils.get(guild.roles, name="Dark Green")
        await member.add_roles(role)
    if emoji == '🟪':
        role = discord.utils.get(guild.roles, name="Purple")
        await member.add_roles(role)
    if emoji == '💜':
        role = discord.utils.get(guild.roles, name="Light Purple")
        await member.add_roles(role)
    if emoji == '🟣':
        role = discord.utils.get(guild.roles, name="Dark Purple")
        await member.add_roles(role)
    if emoji == '🐌':
        role = discord.utils.get(guild.roles, name="Dark Pink")
        await member.add_roles(role)
    if emoji == '🦑':
        role = discord.utils.get(guild.roles, name="Pink")
        await member.add_roles(role)


@bot.event
async def on_raw_reaction_remove(payload):
    guild = await(bot.fetch_guild(payload.guild_id))
    emoji = payload.emoji.name
    if emoji == '🔺':
        role = discord.utils.get(guild.roles, name="Light Red")
    if emoji == '🟥':
        role = discord.utils.get(guild.roles, name="Red")
    if emoji == '🔴':
        role = discord.utils.get(guild.roles, name="Dark Red")
    if emoji == '🟦':
        role = discord.utils.get(guild.roles, name="Blue")
    if emoji == '🔵':
        role = discord.utils.get(guild.roles, name="Dark Blue")
    if emoji == '🔷':
        role = discord.utils.get(guild.roles, name="Light Blue")
    if emoji == '🟧':
        role = discord.utils.get(guild.roles, name="Orange")
    if emoji == '🔶':
        role = discord.utils.get(guild.roles, name="Light Orange")
    if emoji == '🟠':
        role = discord.utils.get(guild.roles, name="Dark Orange")
    if emoji == '💛':
        role = discord.utils.get(guild.roles, name="Light Yellow")
    if emoji == '🟨':
        role = discord.utils.get(guild.roles, name="Yellow")
    if emoji == '🟡':
        role = discord.utils.get(guild.roles, name="Dark Yellow")
    if emoji == '🟩':
        role = discord.utils.get(guild.roles, name="Green")
    if emoji == '🍏':
        role = discord.utils.get(guild.roles, name="Light Green")
    if emoji == '🟢':
        role = discord.utils.get(guild.roles, name="Dark Green")
    if emoji == '🟪':
        role = discord.utils.get(guild.roles, name="Purple")
    if emoji == '💜':
        role = discord.utils.get(guild.roles, name="Light Purple")
    if emoji == '🟣':
        role = discord.utils.get(guild.roles, name="Dark Purple")
    if emoji == '🐌':
        role = discord.utils.get(guild.roles, name="Dark Pink")
    if emoji == '🦑':
        role = discord.utils.get(guild.roles, name="Pink")
    member = await(guild.fetch_member(payload.user_id))
    if member is not None:
        await member.remove_roles(role)
    else:
        print("member not found")


bot.run(token)
