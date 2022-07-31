#spreadsheet 
# import schedule
from time import sleep
import sys
import datetime, time
import pandas as pd
from pybit import usdt_perpetual
import gspread
import json
from oauth2client.service_account import ServiceAccountCredentials
import bybit, requests

# 0724 → 0728 少し前倒しで処理を開始(実現実益→未実現損益・「:00」きっかりに注文だせるように)
# 結論：未実現損益さえ取得出来ればどっちで注文だせるか確認できる→すぐ注文できる

# 【　設定　】
client = bybit.bybit(test=False, api_key="aGeCDjuKFzC3HWDvsm", api_secret="s1cXenmamcIdlWhJZZzUnVuPeWHZQxxMCCFG")
# Lot = 0.001 # test
Lot = 0.05  # 本番

#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
#　<1> 未実現損益を取得
#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
import bybit

# 現在のポジション
myposition = client.LinearPositions.LinearPositions_myPosition(symbol="BTCUSDT").result()
# サイズ
Buy_size = myposition["result"][0]["size"]
Sell_size = myposition["result"][1]["size"]
#------------------------------------------------------------------
# サイズのある方から未実現損益を取得
if Buy_size > 0:
    unrealized_pnl = myposition["result"][0]["unrealised_pnl"]
elif Sell_size > 0:
    unrealized_pnl = myposition["result"][1]["unrealised_pnl"]
#-------------------------------------------------------------------------------------------------------------
print(f"【　未実現損益を取得　】: Buy_size:{Buy_size} / Sell_size:{Sell_size} / unrealized_pnl:{unrealized_pnl} ")

#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
#　<2> SSに転記
#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
# 【参考】 https://qiita.com/164kondo/items/eec4d1d8fd7648217935  /  https://tanuhack.com/operate-spreadsheet/#Python
#------------------------------------------------------------------------------------------------------------------
from oauth2client.service_account import ServiceAccountCredentials
import gspread

# Google Spread Sheetsにアクセス
def connect_gspread(jsonf,key):
    scope = ['https://spreadsheets.google.com/feeds','https://www.googleapis.com/auth/drive']
    credentials = ServiceAccountCredentials.from_json_keyfile_name(jsonf, scope)
    gc = gspread.authorize(credentials)
    SPREADSHEET_KEY = key
    worksheet = gc.open_by_key(SPREADSHEET_KEY).sheet1
    return worksheet
#------------------------------------------------------------------
# ws
jsonf = "emabot-356208-14f539cc3eb9.json"
spread_sheet_key = "1FPmRKjhI4nOpWK7uI1I54he4CYWVW3zLu462Fn5z_tE"
ws = connect_gspread(jsonf,spread_sheet_key)
#------------------------------------------------------------------
# 転記する行を取得
def next_available_row(sheet1):
    str_list = list(filter(None, sheet1.col_values(1)))
    return str(len(str_list))
#------------------------------------------------------------------
# 【 転記行 】
active_row = next_available_row(ws) 
#------------------------------------------------------------------
# 【 転記 】
A = ws.update_cell(active_row, 17, unrealized_pnl)
#------------------------------------------------------------------
転記シート = A[updatedRange]
            
#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
#　<3> BuySell_flg 取得
#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
# 未実現損益転記した次の行のflag
BS_flg = int(ws.cell(active_row+1, 11).value)
print(f"【　SS転記 】: 転記シート:{転記シート} / BuySell_flg:{BS_flg} ")
print(type(BS_flg))

#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
#　<4> 注文
#■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
