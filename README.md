# sentment_practice

中文情感分析


雖然目標是做出金融領域的情感分析，但找到現在中文字典都是普通用詞的，只好先來紀錄一下
這份辭典我覺得好的地方是有基本的情緒分數，上次使用過貝式分類效果也不甚理想，因此想說那就先從辭典下手吧。
檔案是xlsx檔，有三份工作表Sheet1是總分類 Sheet2是正向詞Sheet3是負向詞
用pandas.read_exel打開
import pandas as pd
sentment_table = pd.read_excel('情感辭典.xlsx')
sentment_table.drop(['Unnamed: 10','Unnamed: 11'],inplace=True,axis=1)
pos_table = pd.read_excel('情感辭典.xlsx','Sheet2')
neg_table = pd.read_excel('情感辭典.xlsx','Sheet3')
然後我把pos、neg兩份轉成dict。再把neg轉呈負數後合併起來
pos_dict = dict(zip(list(pos_table.word),list(pos_table.強度)))
neg_dict = dict(zip(list(neg_table.word),map(lambda a:a*(0-1),list(neg_table.強度)) ))
sentment_dict={**pos_dict,**neg_dict}
將這份字典的單字加進結巴分詞，但我發現結巴不能加入數字，因此需要先轉換，或者先行判斷裡面是否是數字
def is_number(s):
    try:
        float(s)
        return True
    except ValueError:
        pass
 
    try:
        import unicodedata
        unicodedata.numeric(s)
        return True
    except (TypeError, ValueError):
        pass
 
    return False
把不是數字的字典的單字加進結巴分詞
for w in sentment_dict.keys():
    if is_number(w):
        pass
    else:
        jieba.suggest_freq(w,True)
準備斷詞和停止詞
import re
stop_words = [re.findall(r'\S+',w)[0] for w in open('stopwords.txt',encoding='utf8').readlines() if len(re.findall(r'\S+',w))>0]
def sent2word(sentence,stop_words=stop_words):
    words = jieba.cut(sentence, HMM=False)
    words = [w for w in words if w not in stop_words]
    return words
那就利用簡單的函數來執行情感分析吧
def get_sentment(sent):
    tokens = sent2word(sent)
    score = 0
    countword = 0
    for w in tokens:
        
        if w in sentment_dict.keys():
            score += sentment_dict[w]
            countword += 1
    if countword != 0:
        return score/countword
    else:
        return 0


sent ="我很喜歡他們"
get_sentment(sent)
-> 5


sent ="我討厭老鼠"
get_sentment(sent)
-> -7
get_sentment()會得到一個數字，0是中間值，越高越正面，越低越負面
不過這方法就像查表一樣，只單靠辭典來執行
把抓取到有分數的字顯示一下
def get_sentment(sent):
    tokens = sent2word(sent)
    score = 0
    countword = 0
    for w in tokens:        
        if w in sentment_dict.keys():
            score += sentment_dict[w]
            countword += 1
            print(w,end=' ')
            print(sentment_dict[w])
    if countword != 0:
        print(score/countword)
    else:
        return 0


sent ="跟一個低級人說話要那麽講究"
get_sentment(sent)
-->
低級 -7
講究 7
總分數＝ 0.0


反正會以些句子輸出的分數似乎也太不正確
詞性可能也是需要考慮的點，不知道jieba是否可以cover
jieba分詞表
import jieba.posseg as pseg
words = pseg.cut("我不認為人類非常聰明")
for word, flag in words:
    print('%s %s' % (word, flag))
--> 
我 r
不 d
認為 v
人類 n
非常 d
聰明 x
