# 1-Wrapping-Reel
Slot Machine With 1 Reel Set that "Wraps Around" To Fill 5 Reels

# -*- coding: utf-8 -*-
"""
Created on Thu Jun 22 10:20:52 2017
@author: Chris Thomas
"""

This is a slot machine game that uses 1 single reel strip. The strip wraps
around from reel 1-5.

The Base Game has a special feature as described below:

The "Special" Feature is triggered by landing one SF symbol anywhere within
the symbol window. Upon triggering the Butterfly Feature, one of three feature types
is chosen from the ButterflyFeature Table. 
If the MajorUpgrade feature type is chosen, all of the M1, M2, M3, and M4 symbols are
transformed into the M1 symbol, and the SF symbol is transformed into the X3 symbol. 
If the Wild Column feature type is chosen, all of the symbols in the column that the SF
symbol landed in are turned into WW, and any symbols directly adjacent to the left and
right of the SF symbol’s landing position are also turned into WW. 
If the WildRow feature type is chosen, all of the symbols in the row that the SF symbol
landed in are turned into WW. 
If either the MajorUpgrade or WildColumn feature types are chosen and the resulting
playline win total is not more than the pre-feature payline win total, then the symbol
window is reset to the pre-feature configuration, and the WildRow feature occurs.
The Butterfly Feature occurs before the Multiplier Upgrade feature and does not
occur on a bonus-triggering spin.

"The Multiplier Upgrade Feature is triggered on any basegame spin that results in 2 or more
WW anywhere within the symbol window. It should be noted that any wilds generated from the
"Special" Feature count towards the 2 WW qualifier for the Multiplier Upgrade Feature. 
Once triggered, the Multiplier Upgrade Feature replaces each WW within the symbol window
(including those resulting from any "Special" Features) with a symbol pulled from the
MultiplierFeature Table. The replacement symbol is pulled individually for each WW in the
symbol window with replacement. 
The Multiplier Upgrade Feature occurs after the "Special" Feature, and after any triggered bonuses. "



When 3 BN are located on the screen, a weighted table is used to determine which of 2 free game features is used
The pick bonus is simple
The free game is played according to the following rules


"The Free Games Bonus is triggered through the On-Reel Bonus Determination Feature. The player is awarded 8 free games.
Free games are played using the FreeGames Reel Strip and FreeGames Symbol Window. It should be noted that the FreeGames
Reel Strip contains symbols CA through DL, which have credit prize equivalents according to the SymbolLookUp Table. It
should also be noted that the FreeGames symbol BL is a blank symbol with no associated prize.
The FreeGames Symbol Window contains 10 Win Rectangles, each outlining 3 symbol positions. The first Win Rectangle
outlines the topmost 3 symbols in the first column of the FreeGames Symbol Window, and the the second Win Rectangle
outlines the bottommost 3 symbols in the first column of the FreeGames Symbol Window. The third and fourth Win Rectangles
outline the topmost and bottommost 3 symbols in the second column of the FreeGames Symbol Window etc… 
During the free games, Win Rectangles may become activated. Each activated Win Rectangle has an accumulating meter that
automatically starts with 5 credits. After each spin, each activated Win Rectangle collects the awards it contains;
credit values are added to each Win Rectangle’s accumulating meter first, and then the accumulating meter for any activated
Win Rectangle containing an X2 symbol is multiplied by 2.
The two Win Rectangles in the middle column of the FreeGames Symbol Window are automatically activated before the 1st free
spin. Any time the SF symbol lands in an inactive Win Rectangle, that Win Rectangle activates. Landing the SF symbol in an
activated Win Rectangle awards the player an additional free game. The free games bonus is complete when no free games
remain, or after 15 free games have been played. After the final game, all of the accumulating meters from all of the
activated Win Rectangles are awarded to the player.
All credit values in the free games are multiplied by the bet multiplier of the bonus-triggering bet."


```Python
import pandas as pd
import numpy as np
import random as rd
import os

from datetime import datetime
pd.options.mode.chained_assignment = None

#Helper Functions

###############################
```

```Python
def ExtendReel(Reel):
    extra = pd.Series([Reel[i] for i in range(100)])
    Reel = Reel.append(extra)
    
    Reel.index = np.arange(0,len(Reel))
    
    return(Reel)
```    
Extends the reel so that when we get to the last position, the first position on the reel is after it

```Python
def Create_Table_Probs(table):
    temp = [0]*len(table)
    
    for i in range(len(table)):
        temp[i] = table.iloc[i,1]-1 + sum(table.iloc[0:i,1])
    
    table = table.drop(table.columns[1],axis = 1)
    table['Weights'] = temp  
    table.index = np.arange(0,len(table))
      
    return(table)
```
The developer sheet uses weights. This creates a weighted table using cumulative weights which is easier to randomly select from

```Python
def Create_Pays():
    Pay = {}
    Pay[wild] = {5:100,4:35,3:10}
    Pay['M1'] = {5:100,4:35,3:10}
    Pay['M2'] = {5:75,4:35,3:10}
    Pay['M3'] = {5:40,4:25,3:10}
    Pay['M4'] = {5:40,4:25,3:10}
    Pay['F5'] = {5:30,4:15,3:5}
    Pay['F6'] = {5:30,4:15,3:5}
    Pay['F7'] = {5:30,4:15,3:5}
    Pay['F8'] = {5:30,4:15,3:5}
    Pay['F9'] = {5:30,4:15,3:5}
    Pay['F0'] = {5:30,4:15,3:5}

    return(Pay)

#creates dictionary of the lines     
def Create_Line_Dict(Lines):
    LineDict = {}
    
    for line in range(len(Lines)):
        LineDict[line] = {}
        
        for col in range(5):
            LineDict[line][col] = int(Lines.iloc[line,col])

    return(LineDict)
    
#replaces symbols with their credit equivalent
def Replace_Free_Reels(reel,symbols):
    symbols.columns = [0,1]
    symbols.index = [np.arange(0,len(symbols))]

    SymbolDict = {}
    for i in range(len(symbols)):
        SymbolDict[symbols.iloc[i,0]] = symbols.iloc[i,1]

    SymbolDict['BL'] = 0
    
    for i in range(len(reel)):
        try:
            reel[i] = SymbolDict[reel[i]]
        except:
            reel[i] = reel[i]

    return(reel)
    
######################################
#Variables

fly = 'SF' 
wild = 'WW' 
bonus = 'BN'

data = pd.read_excel(os.path.join(os.getcwd(),"Data2.xlsx"), header = None)

Reel = data.iloc[:,0]
ReelLength = len(Reel)
Reel = ExtendReel(Reel)


Lines = data.iloc[0:50,4:9]
#Lines = Create_Line_Dict(Lines)



Reel1 = [10,11,12,13]
Reel2 = [19,18,17,16]
Reel3 = [22,23,24,25]
Reel4 = [31,30,29,28]
Reel5 = [34,35,36,37]


WindowStops = pd.DataFrame({1:Reel1,
                       2:Reel2,
                       3:Reel3,
                       4:Reel4,
                       5:Reel5})

```
Because only 1 reel is used, these represent the visible window locations

```Python
Offset = [0,1,2,3]

MultiplierTable = data.iloc[3:6,16:18]
ButterflyTable = data.iloc[3:6,11:13]


Pay = Create_Pays()
MultTable = Create_Table_Probs(MultiplierTable)
FlyTable = Create_Table_Probs(ButterflyTable)

NumLines = len(Lines)
Lines = Create_Line_Dict(Lines)

#get the intial free reels data
FreeReel = data.iloc[0:5000,1]
FreeSymbols = data.iloc[12:50,11:13]

#change the free reels data to useable data
FreeReelLength = len(FreeReel)
FreeReel = ExtendReel(FreeReel)
FreeReel = Replace_Free_Reels(FreeReel,FreeSymbols)



Reel1 = [9,10,11,12,13,14]
Reel2 = [20,19,18,17,16,15]
Reel3 = [21,22,23,24,25,26]
Reel4 = [32,31,30,29,28,27]
Reel5 = [33,34,35,36,37,38]

FreeWindowStops = pd.DataFrame({1:Reel1,
                            2:Reel2,
                            3:Reel3,
                            4:Reel4,
                            5:Reel5})

FreeOffset = [0,1,2,3,4,5]


Win1 = [[0,0],[1,0],[2,0]]
Win2 = [[3,0],[4,0],[5,0]]

Win3 = [[0,1],[1,1],[2,1]]
Win4 = [[3,1],[4,1],[5,1]]

Win5 = [[0,2],[1,2],[2,2]]
Win6 = [[3,2],[4,2],[5,2]]

Win7 = [[0,3],[1,3],[2,3]]
Win8 = [[3,3],[4,3],[5,3]]

Win9 = [[0,4],[1,4],[2,4]]
Win10 = [[3,4],[4,4],[5,4]]

FreeFramePositions = pd.DataFrame([[Win1,Win3,Win5,Win7,Win9],
                   [Win2,Win4,Win6,Win8,Win10]])    


#########################################################

```
#prints a window which is made using a dictionary
#in a nicer to view format
```Python
def Print_Window(Window):
    df = pd.DataFrame(np.zeros(shape = (4,5)))

    for col in range(5):
        for row in range(4):
            df.iloc[row,col] = Window[col][row]
    #print(df)        
    return(df)  
```
#creates a window using a dictionary for efficiency
#uses the user defined Offset as the stop position    
```Python
def Create_Window(WindowStops,Offset,stop):
    Window = {}
    
    for reel in range(5):
        Window[reel] = {}
        for i in range(len(Offset)):
            row = Offset[i]

            Window[reel][row] = WindowStops.iloc[row,reel] + stop

    return(Window)
```  

#returns window of symbols rather than positions
```Python
def Get_Symbols(Positions,Reel):
    Window = {}
    for col in range(5):
        Window[col] = {}
        for i in range(len(Offset)):
            row = Offset[i]

            Window[col][row] = Reel[Positions[col][row]]

    return(Window)
```

```Python
#returns random stop        
def Get_Stop(ReelLength):
    return(rd.randint(0,ReelLength))

def Get_Line_Pay(LineNum,Lines, Window,wild):
    WildMults = ['1','2','3']
    
    Line = [Lines[LineNum][reel] for reel in range(5)]
    Symbols = [Window[col][Line[col]] for col in range(5)]

    #count wild symbols before first symbol
    WildCount = 0
    SymbolCount = 0
    Mult = 1
    for i in range(5):
        if Symbols[i] == wild or Symbols[i] in WildMults:
            WildCount += 1
            if Symbols[i] in WildMults:
                Mult *= eval(Symbols[i])
        else:           
            break
        
    if WildCount == 5:
        CurrSymbol = wild
        SymbolCount = 5
    else:
            
        for symbol in Symbols[WildCount:5]:
            if SymbolCount ==0:
                CurrSymbol = symbol
                SymbolCount = WildCount + 1
            else:
                if CurrSymbol == symbol or symbol == wild or symbol in WildMults:
                    SymbolCount +=1
                    
                    if symbol in WildMults:
                        Mult*= eval(symbol)
                else:
                    break
                
                
    #SymbolCount += WildCount
    
    SymbolPay = max(Get_Symbol_Pay(CurrSymbol,SymbolCount)*Mult,Get_Symbol_Pay(wild,WildCount)*Mult)
    return(SymbolPay)    

def Get_Window_Pay(Window,NumLines,Lines):
    WindowPay = 0
    for i in range(NumLines):
        WindowPay += Get_Line_Pay(i,Lines,Window,wild)
       
    return(WindowPay)
        
def Get_Symbol_Pay(symbol,count):
    try:
        pay = Pay[symbol][count]
    except:
        pay = 0
    return(pay)

def PlayOnce():    
    global WindowStops, Offset, Reel
    Stop = Get_Stop(ReelLength)
    
    
    Positions = Create_Window(WindowStops, Offset,Stop)
    Window = Get_Symbols(Positions,Reel)
    Print_Window(Window)
    Window = Butterfly(Window)
    Window = Multiplier(Window)
    
    WindowPay = Get_Window_Pay(Window,NumLines,Lines)    
    
    BonusCount = Check_For_Bonus(Window)
    
    if BonusCount >= 3:
        BonusType = rd.randint(0,2)
        if BonusType == 2:
            WindowPay += Play_Free_Game()
        else:
            WindowPay += Pick_Bonus()
    

    return(float(WindowPay))

def PlayABunch(its):
    
    TotalPay = 0    
    for i in range(its):
        TotalPay += PlayOnce()
        
        if i >0:
#            if TotalPay/its/50 >= .6:
#                print(TotalPay/its/50)
            if i % 1000 == 0:
                print('You are ' + str(float(i)/its) + ' done and your RTP is' + str(float(TotalPay)/float(i)/float(50)))
    return(TotalPay/its/50)

```
Counts the number of butterflies, our trigger symbol, in the window
```Python
def Butterfly(Window):
    def Check_Butterfly(Window):
        Index = []
        for col in range(5):
            for i in range(4):
                row = Offset[i]

                if Window[col][row] == fly:
                    Index.append([row,col])
                    
        return(Index)
   ```
   Uses a random number to deterine which feature will be played
   ```Python
    def Butterfly_Type(table):
        MaxRange = table.iloc[2,1]

        weights = table.iloc[:,1]
        
        random = rd.randint(0,MaxRange)
        
        for i in range(len(table)):
            if random <= weights[i]:
                return(table.iloc[i,0])        

            
    def Wild_Row_Feauture(Window,Row):
        for Col in range(5):
            Window[Col][Row] = wild
    
        return(Window)
        
    def Wild_Col_Feauture(Window,Col):
        for Row in range(4):
            Window[Col][Row] = wild
    
        return(Window)
        
    def Major_Upgrade_Feature(Window):
        UpgradeSymbol = 'M1'
        Upgrades = ['M1','M2','M3','M4']
    
        for col in range(5):
            for i in range(4):
                row = Offset[i]
    
                if Window[col][row] in Upgrades:
                    Window[col][row] = UpgradeSymbol
                elif Window[col][row] == fly:
                    Window[col][row] = '3'
                    
        return(Window)
        
     
        
    Index = Check_Butterfly(Window)
     #if not the index is not emptry, proceed  
    if Index:
        for index in Index:
            FlyType = Butterfly_Type(FlyTable)
            
            if FlyType == 'MajorUpgrade':
                Window = Major_Upgrade_Feature(Window)
            elif FlyType == 'WildColumn':
                Window = Wild_Col_Feauture(Window,index[1])
            elif FlyType == 'WildRow':
                Window = Wild_Row_Feauture(Window, index[0])
                
            return(Window)
    else:
        return(Window)
        
        
def Multiplier(Window):
    def Check_Mult(Window):
        
        Index = []
        for col in range(5):
            for i in range(4):
                row = Offset[i]

                if Window[col][row] == wild:
                    Index.append([row,col])
                    
        return(Index)
                
    def Mult_Type(table):
        MaxRange = table.iloc[2,1]

        weights = table.iloc[:,1]
        
        random = rd.randint(0,MaxRange)
        
        for i in range(len(table)):
            if random <= weights[i]:
                MultType = table.iloc[i,0]
                break

        if MultType == 'WW':
            return('1')
        elif MultType =='X2':
            return('2')
        elif MultType == 'X3':
            return('3')
        
        
    Index = Check_Mult(Window)
     #if not the index is not emptry, proceed  
    if len(Index) >=2 :
        for index in Index:
            MultType = Mult_Type(MultTable)
            Window[index[1]][index[0]] = MultType            
            
        return(Window)
    else:
        return(Window)
        
        

#Pick Bonus
def Pick_Bonus():
    LowCredits = data.iloc[3:11,22:24]
    MediumCredits = data.iloc[3:14,27:29]
    HighCredits = data.iloc[3:16,32:34]
    CreditPick = data.iloc[3:15,37]
    
    Round1 = data.iloc[3:7,42]
    Round2 = data.iloc[3:7,47]
    Round3 = data.iloc[3:7,52]
    Round4 = data.iloc[3:7,57]
    Round5 = data.iloc[3:7,62]
    Round6 = data.iloc[3:7,67]
    Round7 = data.iloc[3:7,72]
    Round8 = data.iloc[3:7,77]
    Round9 = data.iloc[3:7,82]
    Round10 = data.iloc[3:7,87]

    list = [LowCredits,MediumCredits,HighCredits,CreditPick,
            Round1,Round2,Round3,Round4,Round5,
            Round6,Round7,Round8,Round9,Round10]
            
    for item in list:
        item.index = np.arange(0,len(item))
    
    Cont = True
    Credit = 0
    RoundLevel = 1
    while Cont == True:
        #determine pick level and remove from futures possibilities
        PickLevelNum = rd.randint(0,len(CreditPick)-1)
        #print(str(PickLevelNum) + 'Pick Level Num')
        PickType = CreditPick[PickLevelNum]
        #print(str(PickType) + 'Pick Type')
        CreditPick.drop(CreditPick.index[PickLevelNum], inplace = True)
        CreditPick.index = np.arange(0,len(CreditPick))
        #print(CreditPick)
        
        #determines actual pick and removes from table
        pick = eval(PickType)
        PickProb = Create_Table_Probs(pick)
        MaxRange = PickProb['Weights'][len(PickProb)-1]
        PickNum = rd.randint(0,MaxRange)
        #print(str(PickNum) + 'Pick Num')
        for i in range(len(PickProb)):
            if PickNum <= PickProb['Weights'][i]:
                CurrentCredit =  PickProb.iloc[i,0]
                #print(str(PickProb.iloc[i,0]) + 'Credit Val')
                PickProb.drop(i, inplace = True)
                PickProb.index = np.arange(0,len(PickProb))
                break
        
                
        #print(Credit)
        #Choose Multiplier
        if RoundLevel == 10:
            break
        else:
            MultLevel = rd.randint(0,3)
            MultTable = eval('Round' + str(RoundLevel))
            Multiplier = MultTable[MultLevel]
    
            if Multiplier != 1:
                #print(Multiplier)
                CurrentCredit*=Multiplier
            else:
                Cont = False
    
            RoundLevel += 1
            Credit += CurrentCredit
            #print(str(RoundLevel)+ ' Round Level')
            
#        print('You picked' + str(PickType) + ' so you got ' + str(Credit)  
#        + 'and you reached level' + str(RoundLevel))
        
    #print(Credit)    
    return(Credit)

```
It was easier to just copy the functions than use the same functions
```Python

#Free Game Funnctions
##########################

def Check_For_Bonus(Window):    
    BonusCount = 0
    for col in range(5):
        for i in range(4):
            row = Offset[i]
            if Window[col][row] == bonus:
                BonusCount += 1
                    
    return(BonusCount)

def Initialize_Free_Frame() :
    Matrix = [[[] for col in range(5)] for row in range(2)]
    for col in range(5):
        for row in range(2):
            temp = {}
#            temp['Syms'] = Dict_Slice(FreeWindow,df.iloc[row,col])
            Matrix[row][col] = temp
            Matrix[row][col]['Active'] = 0
            Matrix[row][col]['Mult'] = 1
            Matrix[row][col]['Cred'] = 5

    for col in [2]:
        for row in [0,1]:
            Matrix[row][col]['Active'] = 1

    return(Matrix)
    
def Update_Free_Frame(frame,FreeWindowStops):
    stop = Get_Stop(FreeReelLength)
    
    #Matrix = [[[] for col in range(5)] for row in range(2)]
    for reel in range(5):
        
        for i in [0,1]:
            temp = [0]*3
            for j in range(3):
                row = FreeOffset[i*3+j]
    
                SymbolPosition = FreeWindowStops.iloc[row,reel] + stop
                temp[j] = FreeReel[SymbolPosition]

    
            #Matrix[i][reel] = temp
            frame[i][reel]['Syms']= temp

    #we should have a dictionary with the multiplier, credits and active status
    #as well as the current symbols

    return(frame)
        
def Play_Free_Window(FreeFrame,FreeCount):    
    #FreeCount = 0
    for col in range(5):
        for row in range(2):
#            print(str(col) + ',' + str(row))
            frame = FreeFrame[row][col]


            if frame['Active'] == 1:
                if 'X2' in frame['Syms']:
                    frame['Mult'] *=2
                    frame['Syms'].remove('X2')
                if fly in frame['Syms']:
                    FreeCount += 1
                    frame['Syms'].remove(fly)
                
                frame['Cred'] += sum(frame['Syms'])
                frame['Cred'] *= frame['Mult']

                
                frame['Mult'] = 1
                
                
            else:
                if fly in frame['Syms']:
                    frame['Active'] = 1
                    frame['Syms'].remove(fly)
                    
                    
                    #not sure if this should be included or not
                    if 'X2' in frame['Syms']:
                        frame['Mult'] *=2
                        frame['Syms'].remove('X2')
                    
                    frame['Cred'] += sum(frame['Syms'])
                    frame['Cred'] *= frame['Mult']

                
                    frame['Mult'] = 1                    
    return([FreeFrame,FreeCount])        
        
def Play_Free_Game():
    Window = Initialize_Free_Frame()
    
    FreeCount = 8
    while FreeCount >0:        
        Window = Update_Free_Frame(Window,FreeWindowStops)
        Window,FreeCount = Play_Free_Window(Window,FreeCount) 
        FreeCount -=1 

    Pay = 0
    for reel in range(5):
        for row in range(2):
            
            if Window[row][reel]['Active'] == 1:
                Pay += Window[row][reel]['Cred']
        
    return(Pay)
   
#########################
#Functions for debugging

def TestPlayOnce(Stop): 
    
    global WindowStops, Offset, Reel
    #Stop = Get_Stop(ReelLength)
    
    
    Positions = Create_Window(WindowStops, Offset,Stop)
    Window = Get_Symbols(Positions,Reel)    
    Window = Butterfly(Window)
    Window = Multiplier(Window)
    
    Print_Window(Window)
    
    WindowPay = Get_Window_Pay(Window,NumLines,Lines)
    print(WindowPay)
    
    
    BonusCount = Check_For_Bonus(Window)
    
    if BonusCount >= 3:
        BonusType = rd.randint(0,2)
        if BonusType == 2:
            WindowPay += Play_Free_Game()
        else:
            WindowPay += Pick_Bonus()
    
    print(WindowPay)
    return(float(WindowPay))
    
def Play_Only_Base(its):
    global WindowStops, Offset, Reel
    TotalPay = 0
    
    for i in range(its):
        Stop = Get_Stop(ReelLength)
        
        
        Positions = Create_Window(WindowStops, Offset,Stop)
        Window = Get_Symbols(Positions,Reel)    
        Window = Butterfly(Window)
        Window = Multiplier(Window)
        
        WindowPay = Get_Window_Pay(Window,NumLines,Lines)
        
        TotalPay += WindowPay
        if i >0:
            print(TotalPay/50/i)
    print('You are ' + str(float(i)/its) + ' done and your RTP is' + str(float(TotalPay)/float(i)/float(50)))
    
    
    return(float(WindowPay))
    
def Play_Only_Bonus(its):
    global WindowStops, Offset, Reel
    
    WindowPay = 0
    for i in range(its):
        Stop = Get_Stop(ReelLength)
       
        Positions = Create_Window(WindowStops, Offset,Stop)
        Window = Get_Symbols(Positions,Reel)  
        
        BonusCount = Check_For_Bonus(Window)
        
        
        if BonusCount >= 3:
            BonusType = rd.randint(0,2)
            if BonusType == 2:
                WindowPay += Play_Free_Game()
            else:
                WindowPay += Pick_Bonus()
    WindowPay = WindowPay/50/its        
    print(WindowPay)    
    return(float(WindowPay))
    
def Bonus_Hit_Count(its):
    global WindowStops, Offset, Reel
    
    BonusHitsCount = 0 
    for i in range(its):
        Stop = Get_Stop(ReelLength)
        
        
        Positions = Create_Window(WindowStops, Offset,Stop)
        Window = Get_Symbols(Positions,Reel)    
    
        
        BonusCount = Check_For_Bonus(Window)
        if BonusCount >= 3:
            BonusHitsCount += 1
        
    print(BonusHitsCount)        
    return(BonusHitsCount)
    
def Pick_Bonus_Return(its):
    #global WindowStops, Offset, Reel
    
    #BonusHitsCount = 0 
    TotalWin = 0
    for i in range(its):
#        Stop = Get_Stop(ReelLength)
        
        
#        Positions = Create_Window(WindowStops, Offset,Stop)
#        Window = Get_Symbols(Positions,Reel)    
    
        
        #BonusCount = Check_For_Bonus(Window)
        #if BonusCount >= 3:
        TotalWin += Pick_Bonus()
        print(TotalWin)
        
                
        TotalWin = TotalWin/its
    #print(TotalWin)        
    return(TotalWin)
