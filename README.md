# TicTacToe-Game - discord.py
a command for TicTacToe Game
```py
from asyncio import TimeoutError

from discord.ext.commands import BadArgument
from discord.ext.commands import Context
from discord import Member

@client.command(aliases=['tic-tac-toe', 'xo', 'tictactoe'], help='''
    1) The game is played on a grid that's 3 squares by 3 squares.
    2) You are `X`, your friend is `O`. Players take turns putting their marks in empty squares.
    3) The first player to get **3** of her marks in a row (up, down, across, or diagonally) is the winner.
    4) When all 9 squares are full, the game is over.''')
async def TicTacToe(ctx: Context, player2: Member):
    if player2 == ctx.author:
        raise BadArgument('You can enter the name of each player once.')

    t = [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
    e0 = '🔘'
    e1 = '❌'
    e2 = '⭕'
    all_reactions = {
        '⏺': (1, 1), '⬆': (0, 1), '↗': (0, 2),
        '➡': (1, 2), '↘': (2, 2), '⬇': (2, 1),
        '↙': (2, 0), '⬅': (1, 0), '↖': (0, 0),
        '⏹': 'break'
    }

    def write(table) -> str:
        r = ''
        for rew in table:
            for j in rew:
                if j == 0:
                    r += e0
                elif j == 1:
                    r += e1
                else:
                    r += e2
            r += '\n'
        return r[:-1]

    msg = await ctx.send(write(t))
    for i in all_reactions:
        await msg.add_reaction(i)

    for i in range(9):
        play_now = ctx.author if i % 2 == 0 else player2
        try:
            reaction, user = (await client.wait_for('reaction_add', timeout=10, check=lambda r, u:
            r.message == msg and str(r) in list(all_reactions) and u == play_now))
        except TimeoutError:
            await msg.reply(f'Timeout {play_now.mention}')
            break
        del user

        if all_reactions[str(reaction)] == 'break':
            await ctx.send(f'Player {play_now.mention} left from game.')
            break

        else:
            xs = all_reactions[str(reaction)]
            t[xs[0]][xs[1]] = (i % 2) + 1
            all_reactions.pop(str(reaction))

            if any([t[0][0] == t[0][1] == t[0][2] != 0,
                    t[1][0] == t[1][1] == t[1][2] != 0,
                    t[2][0] == t[2][1] == t[2][2] != 0,
                    t[0][0] == t[1][0] == t[2][0] != 0,
                    t[0][1] == t[1][1] == t[2][1] != 0,
                    t[0][2] == t[1][2] == t[2][2] != 0,
                    t[0][0] == t[1][1] == t[2][2] != 0,
                    t[2][0] == t[1][1] == t[0][2] != 0]):
                await msg.edit(content=write(t))
                await msg.reply(f'Player {play_now.mention} is winner')
                break

            await msg.edit(content=write(t))
    else:
        await msg.reply(f'We have not a winner!')
```
