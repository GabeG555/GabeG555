- 👋 Hi, I’m @GabeG555
- 👀 I’m interested in photography and animation
- 🌱 I’m currently learning basic JavaScript and Python (I'm in 9th Grade)
- 💞️ I’m looking to collaborate on nothing
- 📫 How to reach me GitHub!

<!---
GabeG555/GabeG555 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
app.background = gradient('black', 'limeGreen', start='top')
app.stepsPerSecond = 100

app.steps = 0
app.stepHeight = 250

player = Group(
    Circle(190, 315, 5, fill='limeGreen', border='black'),
    Circle(210, 315, 5, fill='limeGreen', border='black'),
    Circle(190, 345, 5, fill='limeGreen', border='black'),
    Circle(210, 345, 5, fill='limeGreen', border='black'),
    Rect(190, 310, 20, 40, fill='limeGreen', border='black'),
    Rect(185, 315, 30, 30, fill='limeGreen', border='black'),
    Rect(187, 312, 26, 36, fill='limeGreen'),
    )
player.dy = 0
player.isJumping = False
player.hitBlock = False
player.tooHigh = False

floor = Rect(0, 350, 400, 50, fill=rgb(35, 35, 35))

blocks = Group()
groundBlocks = Group()
groundBlocks.dy = 0

lava = Group(
    Rect(0, 400, 400, 400, fill=gradient('limeGreen', 'black', start='top'), opacity=90),
    Rect(0, 395, 400, 10,
         fill='black')
    )

score = Label(0, 350, 30, fill='purple', size=30)

instructions = Group(
    Rect(0, 0, 400, 400, fill=app.background),
    Label('Hold the right and left keys to move', 200, 150, size=20),
    Label('Press the up key to jump', 200, 175, size=20),
    Label('Try to jump on top of blocks', 200, 200, size=20),
    Label('Avoid getting hit by blocks', 200, 225, size=20),
    Label('Press space to start', 200, 250, size=20)
    )

def gameOver():
    Rect(80, 80, 240, 150, fill='black')
    Label('Game Over!', 200, 120,fill='limeGreen', size=30)
    Label('Score: ' + str(score.value) + ' feet', 200, 200,fill='limeGreen', size=20)
    app.stop()

def makeBlock():
    size = randrange(30, 70)
    x = randrange(0, 360)
    y = (-1) * size
    color = fill='limeGreen'
    newBlock = Group(
        Rect(x + 5, y, size - 10, size, fill=color, border='black'),
        Rect(x, y + 5, size, size - 10, fill=color, border='black'),
        Circle(x + 5, y + 5, 5, fill=color, border='black'),
        Circle(x + size - 5, y + 5, 5, fill=color, border='black'),
        Circle(x + 5, y + size - 5, 5, fill=color, border='black'),
        Circle(x + size - 5, y + size - 5, 5, fill=color, border='black'),
        Rect(x + 3, y + 3, size - 6, size - 6, fill=color),
        Circle(x + size // 2, y // 2, size // 4, fill='black', border='black')
        )
    blocks.add(newBlock)

def checkHitsGroundBlock():
    # If the player hits a block on the ground, it cant move into it.
    if (player.hitsShape(groundBlocks) == True):
        blockRight = groundBlocks.hitTest(player.right, player.centerY)
        blockLeft = groundBlocks.hitTest(player.left, player.centerY)
        if (blockRight != None):
            player.right = blockRight.left
        elif (blockLeft != None):
            player.left = blockLeft.right

def checkHitsFallingBlock():
    for block in blocks:
        if (player.hitsShape(block) == True):
            return True
    return False

def onKeyPress(key):
    # Starts the jumping animation.
    if ((key == 'up') and (player.isJumping == False)):
        player.isJumping = True
        player.dy = -3

    if (key == 'space'):
        instructions.visible = False

def onKeyHold(keys):
    # Moves the players.
    if ('right' in keys):
        player.centerX += 6
    elif ('left' in keys):
        player.centerX -= 6

    # Wraparound.
    if (player.centerX >= 400):
        player.centerX = 0
    elif (player.centerX <= 0):
        player.centerX = 400

    checkHitsGroundBlock()
    checkHitsFallingBlock()

def tryLandedOnBlock():
    # If the player is falling, and slightly inside of and above a landed block,
    # then lands on top of that block.
    for block in groundBlocks:
        if ((player.bottom >= block.top) and (player.bottom - 5 <= block.top) and
            (player.right > block.left) and (player.left < block.right) and
            (player.dy > 0)):
            player.dy = 0
            player.isJumping = False
            player.tooHigh = False
            player.bottom = block.top
            if (abs(app.stepHeight - player.bottom) > 5):
                if (app.stepHeight - player.bottom > 0):
                   groundBlocks.needToMove = abs(app.stepHeight -
                                                 player.bottom)

def onStep():
    # If the instructions are visible, don't do the rest of onStep.
    if (instructions.visible == True):
        return None

    app.steps += 1
    score.value = int(floor.top - player.bottom)

    # Moves the lava.
    lava.centerY -= 0.075
    if ((player.hitsShape(lava) == True) or (player.centerY >= 400)):
        gameOver()

    # Creates new blocks.
    if (app.steps % 75 == 0):
        rand = randrange(0, 2)
        if (rand == 1):
            makeBlock()

    # Jumping animation.
    player.centerY += player.dy
    player.dy += 0.05
    if (player.bottom >= floor.top):
        player.tooHigh = False
        player.isJumping = False
        player.dy = 0
        player.bottom = floor.top
    else:
        tryLandedOnBlock()

    # Moves the screen down if the player gets high enough.
    if ((app.stepHeight - player.bottom > 5) and (player.tooHigh == False)):
        groundBlocks.dy = 15
        player.tooHigh = True

    if (groundBlocks.dy > 0):
        groundBlocks.centerY += groundBlocks.dy
        floor.centerY += groundBlocks.dy
        lava.centerY += groundBlocks.dy
        groundBlocks.dy = 0

    # Moves all the blocks down.
    for block in blocks:
        if (block.hitsShape(player) == True):
            gameOver()

        if ((block.bottom <= floor.top) and
            (groundBlocks.hitsShape(block) == False)):
            block.centerY += 2
        else:
            blocks.remove(block)
            groundBlocks.add(block)

        # Removes any blocks that have fallen off the canvas.
        if (block.top >= 400):
            blocks.remove(block)

    # Removes any blocks that have landed on the ground and are below the canvas.
    for block in groundBlocks:
        if (block.top >= 400):
            groundBlocks.remove(block)
