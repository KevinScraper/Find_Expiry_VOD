from urllib.request import urlopen
from urllib.error import URLError
import json
import time


while True:

    enter = int(input("輸入欲比較UNIX時間戳: "))
    
    if enter < int(time.time()):
        print("小於現在時間, 重新輸入")
        continue
    
    else:
        break

try:
    with urlopen("http://211.76.126.123:9080/stb_menu/CMS/stb_series.php?smartcard=009804206688&file=MainCategory_movie") as url:
        stb_series_data = json.loads(url.read().decode())
        
except URLError:
    print("stb_series.php could not be found")
    
    
def get_product_info(editorialId = None, videoId = None):
    
    try:
        with urlopen("http://106.1.170.24/Order/product_info.php?smc=009804206688&editorialId=" +editorialId+ "&coupon=0&videoType=0&videoId=" +videoId) as url:
            data = json.loads(url.read().decode())
            
    except URLError:
        print("product_info.php could not be found")
        return None
    
    else:
        return data


def get_total_records():
    
    dict_editorials = stb_series_data.get("total_records")
    
    return dict_editorials


def get_VOD_title(editorials_list_number = 0):
    
    dict_editorials = stb_series_data.get("editorials")
    
    list_content = dict_editorials[editorials_list_number]
    
    list_title = list_content["Title"]
    
    return list_title


def get_VOD_id(editorials_list_number = 0):
    
    dict_editorials = stb_series_data.get("editorials")
    
    list_content = dict_editorials[editorials_list_number]
    
    list_id = list_content["id"]
    
    return list_id
    

def get_VOD_videoId(editorials_list_number = 0):
    
    dict_editorials = stb_series_data.get("editorials")
    
    list_content = dict_editorials[editorials_list_number]
    
    list_videoId = list_content["videoId"]
    
    return list_videoId


def get_VOD_expiration(editorials_list_number = 0):
    
    ID = get_VOD_id(editorials_list_number)
    
    videoId = get_VOD_videoId(editorials_list_number)
    
    dict_product_info = get_product_info(ID, videoId)
    
    expiration = dict_product_info["expiration"]
    
    timeArray = time.strptime(expiration, "%Y/%m/%d %H:%M:%S")
    
    timeStamp = int(time.mktime(timeArray))
    
    return timeStamp

#-----------------------------------------------------------------------------------------------------

def compare_VOD_expiration(timestamp = 0):
    
    for editorials_list_number in range(0, get_total_records()):
        
        time.sleep(1)
        
        try:
            
            if get_VOD_expiration(editorials_list_number) > timestamp:
                
                print(get_VOD_title(editorials_list_number))
                
            
            elif get_VOD_expiration(editorials_list_number) <= timestamp:
                
                timeArray = time.localtime(get_VOD_expiration(editorials_list_number))
                
                print(get_VOD_title(editorials_list_number)+ " " 
                    + str(time.strftime('%Y/%m/%d %H:%M:%S', timeArray)))
                
                break
            
        
        except ValueError:
            
            print(get_VOD_title(editorials_list_number) + "------>這部片的expiration欄位格式不符")
        
            continue
        
        except TypeError:
            
            print(get_VOD_title(editorials_list_number) + "------>這部片發生TypeError")
        
            continue

            
compare_VOD_expiration(enter)

