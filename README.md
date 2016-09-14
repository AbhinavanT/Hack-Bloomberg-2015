# Hack-Bloomberg-2015
#Algorithmic Trading Code for 3rd place finish

import socket
import sys
import time
import pandas as pd
import numpy as np
import math
#import matplotlib.pyplot as plt
    
#HOST, PORT = "codebb.cloudapp.net", 17429
#sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
#sock.connect((HOST, PORT))
#user="HaroldandKumar_MA"
#password="actscifilth"  
#data=user + " " + password + "\n"
#output=""
#sock.sendall(data)
numofsecurities=0


def run(*commands):
    data="\n".join(commands)+"\n"
    sock.sendall(data)
    sfile = sock.makefile()
    rline = sfile.readline()
    output=rline
    return output

def subscribe(user, password):
    HOST, PORT = "codebb.cloudapp.net", 17429
    
    data=user + " " + password + "\nSUBSCRIBE\n"

    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

        sock.connect((HOST, PORT))
        sock.sendall(data)
        sfile = sock.makefile()
        rline = sfile.readline()
        while rline:
            print(rline.strip())
            rline = sfile.readline()
    finally:
        sock.close()

#---------OUR CODE------------

temp=run("MY_SECURITIES")
temp=temp.split(" ")   
numofsecurities=(len(temp)-1)/3

      
def portfolio():
    output=run("MY_SECURITIES")
    parseit=output.split(" ")   
    numofsecurities=(len(parseit)-1)/3
    securities = [[0 for x in range(3)] for x in range(numofsecurities)]
    for x in range(0, numofsecurities):
        for y in range(1,4):
            securities[x][y-1]=parseit[3*x+y]
    return securities

stockname=["abc"]
for x in range(0, numofsecurities):
    stockname.append(portfolio()[x][0])

def tickername(index):
    return stockname[index]

def marketdata():
    output=run("SECURITIES")
    parseoutput=output.split(" ")
    
    securitydata = [[0 for x in range(4)] for x in range(numofsecurities)]
    for x in range(0, numofsecurities):
        for y in range(1,5):
            securitydata[x][y-1]=parseoutput[4*x+y]
    return securitydata

def marketdata2():
    output=run("SECURITIES")
    parseoutput=output.split(" ")
    return parseoutput
      
def order(ticker):
    output=run("ORDERS "+ tickername(ticker))
    parseoutput=output.split(" ")
    elements=(len(parseoutput)-1)/4
    orderdata = [[0 for x in range(4)] for x in range(elements)]
    for x in range(0, elements):
        for y in range(1,5):
            orderdata[x][y-1]=parseoutput[4*x+y]
    return orderdata

def myorder():
    output=run("MY_ORDERS")
    parseoutput=output.split(" ")
    elements=(len(parseoutput)-1)/4
    orderdata = [[0 for x in range(4)] for x in range(elements)]
    for x in range(0, elements):
        for y in range(1,5):
            orderdata[x][y-1]=parseoutput[4*x+y]
    return orderdata

def cash():
    output=run("MY_CASH")
    parseoutput=output.split(" ")
    return float(parseoutput[1])

def bid(ticker,price,shares):
    print("BID "+ tickername(ticker) + " " + str((price)) + " " + str((shares)))
    run("BID "+ tickername(ticker) + " " + str((price)) + " " + str((shares)))


def marketbuy(ticker,shares):
    temp=order(ticker)
    shares=int(shares)
    while shares > 0:
        for x in range(0, len(temp)):
            if temp[x][0]=='ASK':
                tobuy=min(shares,int(temp[x][3]))
                shares=shares-int(tobuy)
                bid(ticker,str(float(temp[x][2])+float(temp[x][2])/1000),tobuy)
                temp[x][0]='BID'
                break

def marketbuy2(ticker,value):
    temp=order(ticker)
    while value > 0:
        tobuy=0
        for x in range(0, len(temp)):
            if temp[x][0]=='ASK':
                tobuy=min(math.floor(float(value)/float(temp[x][2])),int(temp[x][3]))
                value=value-float(tobuy)*float(temp[x][2])
                bid(ticker,str(float(temp[x][2])+float(temp[x][2])/1000),int(tobuy))
                temp[x][0]='BID'
                break
        if tobuy==0:
            break

def marketsell(ticker,shares):
    temp=order(ticker)
    shares=int(shares)
    while shares > 0:
        for x in range(0, len(temp)):
            if temp[x][0]=='BID':
                tosell=min(shares,int(temp[x][3]))
                shares=shares-int(tosell)
                ask(ticker,str(float(temp[x][2])-float(temp[x][2])/1000),tosell)
                temp[x][0]='ASK'
                break

                
def ask(ticker,price,shares):
    run("ASK "+ tickername(ticker) + " " + str((price)) + " " + str((shares)))
    print("ASK "+ tickername(ticker) + " " + str((price)) + " " + str((shares)))

def clearbid(ticker):
    run("CLEAR_BID "+ tickername(ticker))
    
def clearallbids():
    for i in range(1, numofsecurities+1):
        clearbid(i)

def clearask(ticker):
    run("CLEAR_ASL "+ tickername(ticker))


def suscribe():
    run("SUSCRIBE")

def unsuscribe():
    run("UNSUBSRCIBE")

def netvalue(ticker,history):
    n=len(history)
    output=[]
    for x in range(0, n):
        output.append(history[x][ticker-1][1])
    return output
    
def dividend(ticker,history):
    n=len(history)
    output=[]
    for x in range(0, n):
        output.append(history[x][ticker-1][2])
    return output

def volatility(ticker,history):
    n=len(history)
    output=[]
    for x in range(0, n):
        output.append(history[x][ticker-1][3])
    return output

def liquidate(ticker):
    temp=portfolio()
    marketsell(ticker,temp[ticker-1][1])

def liquidateall():
    temp=portfolio()
    for x in range(1, numofsecurities+1):
        marketsell(x,temp[x-1][1])
        
def topdivs():
    divs = []
    divssort=[]
    for x in range(0,numofsecurities):
        maxdiv = marketdata()[x][2]
        divs.append(float(maxdiv)+float(maxdiv)/float(1000+x))
        divssort.append(float(maxdiv)+float(maxdiv)/float(1000+x))
    divssort.sort()
    stockstobuy=[]
    for i in range(0,numofsecurities):
        stockstobuy.append(divs.index(divssort[i])+1)
    return stockstobuy
     
def spread(ticker):
    highestbid=order(ticker)[0][2]
    temp=order(ticker)
    for j in range(0, len(temp)):
        if temp[j][0]=='ASK':
            highestask=temp[j][2]
            break
    return (float(highestask)-float(highestbid))/float(highestask)
#-----------------------------Test Code-------------------------

# run("ORDERS AAPL")

#print order(1)
#marketbuy(1,10)
#marketsell(1,10)

#for x in range(1, 11):
#    marketbuy(x,1)
#raise SystemExit(0)
# --------------------------Loop Start------------------------------------
liquidateall()
t=0.3
initialcash=cash()
equalweight=initialcash/4
holdings=[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
actions=[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
pairstatus=1


x = 1
print x
haltcondition=True
markethistory=[marketdata()]
portfoliohistory=[portfolio()]

bidtemp=[]
asktemp=[]
for i in range(1,numofsecurities+1):
    highestbid=order(i)[0][2]
    bidtemp.append(highestbid)
    temp=order(i)
    for j in range(0, len(temp)):
        if temp[j][0]=='ASK':
            highestask=temp[j][2]
            asktemp.append(highestask)
            break
bidhistory=[bidtemp]
askhistory=[asktemp]
time.sleep(t)


#----------------------------------------------------------test-----------------------------------------------------------
haltcondition=True
dividendrank=topdivs()
stockindex1=0
stockindex2=1
factor=14

marketbuy2(dividendrank[stockindex1],cash()/float(2))
marketbuy2(dividendrank[stockindex2],cash()/float(2))
time.sleep(45)
while haltcondition:
    x=x+1
    
    if (x % 2) == 0:
        stock1=dividendrank[stockindex1]
        stock1next=dividendrank[(stockindex1+2)%numofsecurities]
        print x
        print stockindex1
        print (stockindex1+2)%numofsecurities
        print stock1
        print stock1next
        print factor
        print (8+factor*2)

        
        
        shares=portfolio()[stock1-1][1]
        amount=math.floor(float(shares)/float(5))        
        marketsell(stock1,amount)
        marketbuy2(stock1next,cash())
        portfoliohistory.append(portfolio())
        time.sleep(2)
        marketsell(stock1,amount)
        marketbuy2(stock1next,cash())
        portfoliohistory.append(portfolio())
        time.sleep(2)
        marketsell(stock1,amount)
        marketbuy2(stock1next,cash())        
        portfoliohistory.append(portfolio())
        time.sleep(2)
        marketsell(stock1,amount)
        marketbuy2(stock1next,cash())        
        portfoliohistory.append(portfolio())
        time.sleep(2)
        amount=portfolio()[stock1-1][1]
        marketsell(stock1,amount)
        marketbuy2(stock1next,cash())
        stockindex1=(stockindex1+2)%numofsecurities
        for i in range(0, factor):
            portfoliohistory.append(portfolio())
            time.sleep(2)
        
        
    if (x % 2) == 1:
        stock2=dividendrank[stockindex2]
        stock2next=dividendrank[(stockindex2+2)%numofsecurities]
        
        print x
        print stockindex2
        print (stockindex2+2)%numofsecurities
        print stock2
        print stock2next
        print factor
        print (8+factor*2)
        
        shares=portfolio()[stock2-1][1]
        amount=math.floor(float(shares)/float(5))
        marketsell(stock2,amount)
        marketbuy2(stock2next,cash())  
        portfoliohistory.append(portfolio())
        time.sleep(2)
        marketsell(stock2,amount)
        marketbuy2(stock2next,cash())   
        portfoliohistory.append(portfolio())
        time.sleep(2)        
        marketsell(stock2,amount)
        marketbuy2(stock2next,cash())   
        portfoliohistory.append(portfolio())
        time.sleep(2)        
        marketsell(stock2,amount)
        marketbuy2(stock2next,cash())  
        portfoliohistory.append(portfolio())
        time.sleep(2)
        amount=portfolio()[stock2-1][1]
        marketsell(stock2,amount)
        marketbuy2(stock2next,cash())
        stockindex2=(stockindex2+2)%numofsecurities
        
        for i in range(0, factor):
            portfoliohistory.append(portfolio())
            time.sleep(2)
    if x == 13:
        factor = 21

    if x == 22:
        factor = 35
    # --------------------------Algo End
    
    if x == 2000:
        haltcondition=False
    time.sleep(t)

#----------------------------------------------------------test--------------------------------------------------------

# Step 1 Buy first 2 stocks-----------------------------------------------------------

tobuy=topdivs()
tobuy=tobuy[:3]
originaldividendfactor=[]
for i in tobuy:
    marketbuy2(i,equalweight)
    originaldividendfactor.append(portfolio()[i-1][2])
    
    
    
    
time.sleep(30)
currentdividendfactor=[]

for i in tobuy:
    currentdividendfactor.append(portfolio()[i-1][2])


print currentdividendfactor
print originaldividendfactor


    
# Step 2 Wait 1 minute--------------------------------------------------------


# Step 3 Bid on other 5 stocks--------------------------------------------------------
#availablecash=cash()
#for i in range(1,11):
#    highestbid=order(i)[0][2]
#
#    temp=order(i)
#    for j in range(0, len(temp)):
#        if temp[j][0]=='ASK':
#            highestask=temp[j][2]
#            break
#    
#    if holdings[i]==0:
#        price=0.9*float(highestbid)+0.1*float(highestask)
#        bid(i,price,math.floor(availablecash/price))
#        bid(i,float(highestbid)+float(highestbid)/float(100000000),math.floor(float(availablecash)/float(float(highestbid)+float(highestbid)/float(100000000))))
#        holdings[i]=-1
#print myorder()

# Step 4 Remaining--------------------------------------------------------


    



# Continue algorithm
haltcondition=True
while haltcondition:
    x=x+1
    print x
    markethistory.append(marketdata())
    portfoliohistory.append(portfolio())
    bidtemp=[]
    asktemp=[]
    for i in range(1,numofsecurities+1):
        highestbid=order(i)[0][2]
        bidtemp.append(highestbid)
        temp=order(i)
        for j in range(0, len(temp)):
            if temp[j][0]=='ASK':
                highestask=temp[j][2]
                asktemp.append(highestask)
                break
    bidhistory.append(bidtemp)
    askhistory.append(asktemp)
    # --------------------------Algo Start
    if (x % 120) ==60:
        liquidateall()
        availablecash=cash()
        equalweight=availablecash/5
        
        for i in range(1,numofsecurities+1):
            if holdings[i]==0:
                marketbuy2(i,equalweight)

    if (x % 120) ==0:
        liquidateall()
        availablecash=cash()
        equalweight=availablecash/5
        
        for i in range(1,numofsecurities+1):
            if holdings[i]==1:
                marketbuy2(i,equalweight)
    
    # --------------------------Algo End
    
    if x == 10:
        haltcondition=False
    time.sleep(t)
    
    

# --------------------------Loop End-----------------------------


#print history
#print len(history)

# --------------------------Print Stuff--------------------------
#for x in range(0, 10):
#    print netvalue(x,markethistory)
#    print dividend(x,portfoliohistory)
#    print volatility(x,markethistory)


#for x in range(1, 11):
#    marketsell(x,2)









#sock.close()
#----------------------------------csv--------------------------
raise SystemExit(0)
a = np.asarray(csv)
np.savetxt("test.csv", a, delimiter=",")

#run("BID AAPL 50 10")
        
#x = 1
#while True:
#    print "Code finished %d seconds ago!" % (x)
#    x += 1
#    time.sleep(1)
