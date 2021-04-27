import requests
import json
requisicao = requests.get('https://api.binance.com/api/v3/ticker/price')
cotacao = json.loads(requisicao.text)



def bitfinex_btc(): 
    bitFinexTick1 = requests.get("https://api.bitfinex.com/v1/ticker/btcusd")
    return bitFinexTick1.json()['last_price']

bitfinexlivebtc = float(bitfinex_btc())

print ('BITFINEX BTC = U$',bitfinexlivebtc)
print ('BINANCE BTC = U$',cotacao)
