
    VOWEL_COST = 250
    LETTERS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    VOWELS = 'AEIOU'

    class WOFPlayer():

        def __init__(self,name):
            self.name=name
            self.prizeMoney=0
            self.prizes=[]
        
        def addMoney(self,amt):
            self.prizeMoney=self.prizeMoney+amt
        
        def goBankrupt(self):
            self.prizeMoney=0
        
        def addPrize(self,prize):
            self.prizes.append(prize)
      
        def __str__(self):
            return("{} (${})".format(self.name,self.prizeMoney))
 
    class WOFComputerPlayer(WOFPlayer):
    
        SORTED_FREQUENCIES='ZQXJKVBPYGFWMUCLDRHSNIOATE'
    
        def __init__(self,name,difficulty):
            WOFPlayer.__init__(self,name)
            self.difficulty=difficulty
        def smartCoinFlip(self):
            rand = random.randint(1, 10)
            if rand>self.difficulty :
                return True
            else:
                return False
        def getPossibleLetters(self,guessed):
            letters = []
            for char in LETTERS:
                if char in VOWELS and self.prizeMoney < VOWEL_COST:
                    continue
                letters.append(char)
            return letters
        def getMove(self, category,obscurePhrases,guessed):
            gl = self.getPossibleLetters
            if gl ==[] or gl =='pass':
                return 'pass'
            scf = self.smartCoinFlip
            if scf == True:
                return SORTED_FREQUENCIES[-1]
            elif scf ==False:
                return random.choice(SORTED_FREQUENCIES)
    
    import sys
 
    sys.setExecutionLimit(600000)

    import json
    import random
    import time

    LETTERS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    VOWELS  = 'AEIOU'
    VOWEL_COST  = 250


    def getNumberBetween(prompt, min, max):
        userinp = input(prompt)

        while True:
            try:
                n = int(userinp) # try casting to an integer
                if n < min:
                    errmessage = 'Must be at least {}'.format(min)
                elif n > max:
                    errmessage = 'Must be at most {}'.format(max)
                else:
                    return n
            except ValueError: # The user didn't enter a number
                errmessage = '{} is not a number.'.format(userinp)

       
            userinp = input('{}\n{}'.format(errmessage, prompt))

    def spinWheel():
        with open("wheel.json", 'r') as f:
            wheel = json.loads(f.read())
            return random.choice(wheel)

    def getRandomCategoryAndPhrase():
        with open("phrases.json", 'r') as f:
            phrases = json.loads(f.read())

            category = random.choice(list(phrases.keys()))
            phrase   = random.choice(phrases[category])
            return (category, phrase.upper())


    def obscurePhrase(phrase, guessed):
        rv = ''
        for s in phrase:
            if (s in LETTERS) and (s not in guessed):
                rv = rv+'_'
            else:
                rv = rv+s
        return rv


    def showBoard(category, obscuredPhrase, guessed):
        return """
    Category: {}
    Phrase:   {}
    Guessed:  {}""".format(category, obscuredPhrase, ', '.join(sorted(guessed)))

    print('='*15)
    print('WHEEL OF PYTHON')
    print('='*15)
    print('')

    num_human = getNumberBetween('How many human players?', 0, 10)

    human_players = [WOFHumanPlayer(input('Enter the name for human player #{}'.format(i+1))) for i in range(num_human)]

    num_computer = getNumberBetween('How many computer players?', 0, 10)


    if num_computer >= 1:
        difficulty = getNumberBetween('What difficulty for the computers? (1-10)', 1, 10)

    computer_players = [WOFComputerPlayer('Computer {}'.format(i+1), difficulty) for i in range(num_computer)]

    players = human_players + computer_players


    if len(players) == 0:
        print('We need players to play!')
        raise Exception('Not enough players')


    category, phrase = getRandomCategoryAndPhrase()

    guessed = []

    playerIndex = 0


    winner = False

    def requestPlayerMove(player, category, guessed):
        while True: 
            time.sleep(0.1) 

            move = str(player.getMove(category, obscurePhrase(phrase, guessed), guessed))
            move = move.upper() # convert whatever the player entered to UPPERCASE
            if move == 'EXIT' or move == 'PASS':
                return move
            elif len(move) == 1: # they guessed a character
                if move not in LETTERS: # the user entered an invalid letter (such as @, #, or $)
                    print('Guesses should be letters. Try again.')
                    continue
                elif move in guessed: # this letter has already been guessed
                    print('{} has already been guessed. Try again.'.format(move))
                    continue
                elif move in VOWELS and player.prizeMoney < VOWEL_COST: # if it's a vowel, we need to be sure the player has enough
                    print('Need ${} to guess a vowel. Try again.'.format(VOWEL_COST))
                    continue
                else:
                    return move
            else: # they guessed the phrase
                return move
    

    while True:
    player = players[playerIndex]
    wheelPrize = spinWheel()

    print('')
    print('-'*15)
    print(showBoard(category, obscurePhrase(phrase, guessed), guessed))
    print('')
    print('{} spins...'.format(player.name))
    time.sleep(2) # pause for dramatic effect!
    print('{}!'.format(wheelPrize['text']))
    time.sleep(1) # pause again for more dramatic effect!

    if wheelPrize['type'] == 'bankrupt':
        player.goBankrupt()
    elif wheelPrize['type'] == 'loseturn':
        pass # do nothing; just move on to the next player
    elif wheelPrize['type'] == 'cash':
        move = requestPlayerMove(player, category, guessed)
        if move == 'EXIT': # leave the game
            print('Until next time!')
            break
        elif move == 'PASS': # will just move on to next player
            print('{} passes'.format(player.name))
        elif len(move) == 1: # they guessed a letter
            guessed.append(move)

            print('{} guesses "{}"'.format(player.name, move))

            if move in VOWELS:
                player.prizeMoney -= VOWEL_COST

            count = phrase.count(move) # returns an integer with how many times this letter appears
            if count > 0:
                if count == 1:
                    print("There is one {}".format(move))
                else:
                    print("There are {} {}'s".format(count, move))

                # Give them the money and the prizes
                player.addMoney(count * wheelPrize['value'])
                if wheelPrize['prize']:
                    player.addPrize(wheelPrize['prize'])

                # all of the letters have been guessed
                if obscurePhrase(phrase, guessed) == phrase:
                    winner = player
                    break

                continue # this player gets to go again

            elif count == 0:
                print("There is no {}".format(move))
        else: # they guessed the whole phrase
            if move == phrase: # they guessed the full phrase correctly
                winner = player

                # Give them the money and the prizes
                player.addMoney(wheelPrize['value'])
                if wheelPrize['prize']:
                    player.addPrize(wheelPrize['prize'])

                break
            else:
                print('{} was not the phrase'.format(move))

    # Move on to the next player (or go back to player[0] if we reached the end)
    playerIndex = (playerIndex + 1) % len(players)

    if winner:
    # In your head, you should hear this as being announced by a game show host
    print('{} wins! The phrase was {}'.format(winner.name, phrase))
    print('{} won ${}'.format(winner.name, winner.prizeMoney))
    if len(winner.prizes) > 0:
        print('{} also won:'.format(winner.name))
        for prize in winner.prizes:
            print('    - {}'.format(prize))
    else:
    print('Nobody won. The phrase was {}'.format(phrase))
