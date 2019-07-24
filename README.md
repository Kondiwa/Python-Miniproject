# Python-Miniproject 1
The objective of this miniproject is to exercise your ability to wrangle tabular data set and aggregate large data sets into meaningful summary statistics. We will be working with the same medical data used in the pw miniproject, but will be leveraging the power of Pandas to more efficiently represent and act on our data.

In [1]:
%matplotlib inline
import matplotlib
import seaborn as sns
matplotlib.rcParams['savefig.dpi'] = 144
In [2]:
from static_grader import grader

Introduction
The objective of this miniproject is to exercise your ability to wrangle tabular data set and aggregate large data sets into meaningful summary statistics. We will be working with the same medical data used in the pw miniproject, but will be leveraging the power of Pandas to more efficiently represent and act on our data.

Downloading the data
We first need to download the data we'll be using from Amazon S3:
In [3]:
!mkdir dw-data
!wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/201701scripts_sample.csv.gz -nc -P ./dw-data/
!wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/201606scripts_sample.csv.gz -nc -P ./dw-data/
!wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/practices.csv.gz -nc -P ./dw-data/
!wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/chem.csv.gz -nc -P ./dw-data/

mkdir: cannot create directory ‘dw-data’: File exists
File ‘./dw-data/201701scripts_sample.csv.gz’ already there; not retrieving.

File ‘./dw-data/201606scripts_sample.csv.gz’ already there; not retrieving.

File ‘./dw-data/practices.csv.gz’ already there; not retrieving.

File ‘./dw-data/chem.csv.gz’ already there; not retrieving.


Loading the data
Similar to the PW miniproject, the first step is to read in the data. The data files are stored as compressed CSV files. You can load the data into a Pandas DataFrame by making use of the gzip package to decompress the files and Panda's read_csv methods to parse the data into a DataFrame. You may want to check the Pandas documentation for parsing CSV files for reference.
For a description of the data set please, refer to the PW miniproject. Note that all questions make use of the 2017 data only, except for Question 5 which makes use of both the 2017 and 2016 data.
In [4]:
import pandas as pd
import numpy as np
import gzip
In [5]:
# load the 2017 data

with gzip.open('./dw-data/201701scripts_sample.csv.gz', 'r') as f_csv:
    scripts =pd.read_csv(f_csv)
    scripts.head()
In [6]:
# load the 2016 data
with gzip.open('./dw-data/201606scripts_sample.csv.gz', 'r') as f_csv:
    scripts16 = pd.read_csv(f_csv)
    scripts.head()
In [7]:
col_names=[ 'code', 'name', 'addr_1', 'addr_2', 'borough', 'village', 'post_code']
with gzip.open('./dw-data/practices.csv.gz', 'r') as f_csv:
    practices = pd.read_csv(f_csv,names=col_names)
    practices.head()
In [8]:
with gzip.open('./dw-data/chem.csv.gz', 'r') as g_csv:
    chem = pd.read_csv(g_csv)
    chem.head()

Now that we've loaded in the data, let's first replicate our results from the PW miniproject. Note that we are now working with a larger data set so the answers will be different than in the PW miniproject even if the analysis is the same.

Question 1: summary_statistics
In the PW miniproject we first calculated the total, mean, standard deviation, and quartile statistics of the 'items', 'quantity'', 'nic', and 'act_cost' fields. To do this we had to write some functions to calculate the statistics and apply the functions to our data structure. The DataFrame has a describe method that will calculate most (not all) of these things for us.
Submit the summary statistics to the grader as a list of tuples: [('act_cost', (total, mean, std, q25, median, q75)), ...]
In [9]:
summary_stats = [('items',(8888304,9.133135976111625,29.204198,1.000000,2.000000,6.000000)) , ('quantity', (721457006,741.329835,3665.426958,28.000000,100.000000,350.000000)), ('nic',(71100424.84000002,73.058915,188.070257,7.800000,22.640000,65.000000)) , ('act_cost',(66164096.11999999,67.986613,174.401703,7.330000,21.220000,60.670000))]
In [10]:
grader.score.dw__summary_statistics(summary_stats)

==================
Your score:  1.0
==================

Question 2: most_common_item
We can also easily compute summary statistics on groups within the data. In the pw miniproject we had to explicitly construct the groups based on the values of a particular field. Pandas will handle that for us via the groupby method. This process is detailed in the Pandas documentation.
Use groupby to calculate the total number of items dispensed for each 'bnf_name'. Find the item with the highest total and return the result as [(bnf_name, total)].
In [11]:
grouped=scripts.groupby('bnf_name')['items'].sum()
max(grouped.items(), key = lambda x: x[1])
Out[11]:
('Omeprazole_Cap E/C 20mg', 218583)
In [12]:
most_common_item = [max(grouped.items(), key = lambda x: x[1])]
In [14]:
grader.score.dw__most_common_item(most_common_item)

==================
Your score:  1.0
==================

Question 3: items_by_region
Now let's find the most common item by post code. The post code information is in the practices DataFrame, and we'll need to merge it into the scripts DataFrame. Pandas provides extensive documentation with diagrammed examples on different methods and approaches for joining data. The merge method is only one of many possible options.
Return your results as a list of tuples (post code, item name, amount dispensed as % of total). Sort your results ascending alphabetically by post code and take only results from the first 100 post codes.
NOTE: Some practices have multiple postal codes associated with them. Use the alphabetically first postal code. Note some postal codes may have multiple 'bnf_name' with the same prescription rate for the maximum. In this case, take the alphabetically first 'bnf_name' (as in the PW miniproject).
In [15]:
practices = practices.sort_values('post_code') #Sort to arrange post_code by alphabets
practices = practices[~practices.duplicated(["code"])] 
merged_df = scripts.merge(practices, left_on='practice', right_on='code')
                                             
In [16]:
scripts_df = pd.DataFrame(scripts)
practices_df = pd.DataFrame(practices)
postal = practices_df.sort_values(by=['code','post_code'],ascending=True)
postal.reset_index(inplace=True,drop=True)
postal = postal.drop_duplicates(subset='code', keep="first")
df = pd.merge(scripts_df,postal,left_on=['practice'],right_on=['code'],how='left')
df.head()
Out[16]:

practice
bnf_code
bnf_name
items
nic
act_cost
quantity
code
name
addr_1
addr_2
borough
village
post_code
0
N85639
0106020C0
Bisacodyl_Tab E/C 5mg
1
0.39
0.47
12
N85639
GP OOH VCH
VICTORIA CENTRAL HOSPITAL
MILL LANE
WALLASEY
NaN
CH44 5UF
1
N85639
0106040M0
Movicol Plain_Paed Pdr Sach 6.9g
1
4.38
4.07
30
N85639
GP OOH VCH
VICTORIA CENTRAL HOSPITAL
MILL LANE
WALLASEY
NaN
CH44 5UF
2
N85639
0301011R0
Salbutamol_Inha 100mcg (200 D) CFF
1
1.50
1.40
1
N85639
GP OOH VCH
VICTORIA CENTRAL HOSPITAL
MILL LANE
WALLASEY
NaN
CH44 5UF
3
N85639
0304010G0
Chlorphenamine Mal_Oral Soln 2mg/5ml
1
2.62
2.44
150
N85639
GP OOH VCH
VICTORIA CENTRAL HOSPITAL
MILL LANE
WALLASEY
NaN
CH44 5UF
4
N85639
0401020K0
Diazepam_Tab 2mg
1
0.16
0.26
6
N85639
GP OOH VCH
VICTORIA CENTRAL HOSPITAL
MILL LANE
WALLASEY
NaN
CH44 5UF
In [17]:
df = df[['post_code','bnf_name','items','quantity','act_cost','nic', 'bnf_code', 'practice','addr_1', 'addr_2', 'borough', 'name','village']]
df = df.sort_values('post_code')
df.reset_index(drop = True, inplace = True)
In [18]:
prueba = df.groupby(['post_code','bnf_name']).sum().sort_values('items', ascending=False).sort_index(level='post_code', sort_remaining=False)
prueba.head()
Out[18]:


items
quantity
act_cost
nic
post_code
bnf_name




B11 4BW
Salbutamol_Inha 100mcg (200 D) CFF
706
1082
1511.26
1623.0
Paracet_Tab 500mg
451
44704
918.91
979.1
Metformin HCl_Tab 500mg
387
55828
1593.98
1714.7
Lansoprazole_Cap 30mg (E/C Gran)
385
17500
648.47
693.8
Amoxicillin_Cap 500mg
350
7026
398.77
425.2
In [19]:
prueba.reset_index(level=1,drop=False,inplace=True)
prueba.head()
Out[19]:

bnf_name
items
quantity
act_cost
nic
post_code





B11 4BW
Salbutamol_Inha 100mcg (200 D) CFF
706
1082
1511.26
1623.0
B11 4BW
Paracet_Tab 500mg
451
44704
918.91
979.1
B11 4BW
Metformin HCl_Tab 500mg
387
55828
1593.98
1714.7
B11 4BW
Lansoprazole_Cap 30mg (E/C Gran)
385
17500
648.47
693.8
B11 4BW
Amoxicillin_Cap 500mg
350
7026
398.77
425.2
In [20]:
import numpy as np
post_codes = np.unique(prueba.index.values)
len(post_codes)
Out[20]:
259
In [21]:
answer = df.pivot_table(index='post_code',values='items',aggfunc='sum')
In [22]:
d = []
for i in post_codes:
    d.append({'post_code': i, 'bnf_name': (prueba.loc[i]).iloc[0,:][0], 'items':
              (prueba.loc[i]).iloc[0,:][1]})
    lista = pd.DataFrame(d)
    lista.head()              
In [23]:
mezcla = pd.merge(lista,answer,left_on='post_code',right_index=True ,how='inner')
mezcla.head()
Out[23]:

bnf_name
items_x
post_code
items_y
0
Salbutamol_Inha 100mcg (200 D) CFF
706
B11 4BW
22731
1
Paracet_Tab 500mg
425
B12 9LP
17073
2
Salbutamol_Inha 100mcg (200 D) CFF
556
B18 7AL
20508
3
Metformin HCl_Tab 500mg
1033
B21 9RY
31027
4
Lansoprazole_Cap 30mg (E/C Gran)
599
B23 6DJ
28011
In [24]:
mezcla['share'] = mezcla['items_x'].div(mezcla['items_y'])
mezcla.head()
Out[24]:

bnf_name
items_x
post_code
items_y
share
0
Salbutamol_Inha 100mcg (200 D) CFF
706
B11 4BW
22731
0.031059
1
Paracet_Tab 500mg
425
B12 9LP
17073
0.024893
2
Salbutamol_Inha 100mcg (200 D) CFF
556
B18 7AL
20508
0.027111
3
Metformin HCl_Tab 500mg
1033
B21 9RY
31027
0.033294
4
Lansoprazole_Cap 30mg (E/C Gran)
599
B23 6DJ
28011
0.021384
In [25]:
max_item_by_post = list(zip(mezcla['post_code'],mezcla['bnf_name'].values,mezcla['share'].values))
In [26]:
len(max_item_by_post[0:100])
Out[26]:
100
In [27]:
items_by_region = max_item_by_post[0:100]
In [28]:
grader.score.dw__items_by_region(items_by_region)

==================
Your score:  1.0
==================

Question 4: script_anomalies
Drug abuse is a source of human and monetary costs in health care. A first step in identifying practitioners that enable drug abuse is to look for practices where commonly abused drugs are prescribed unusually often. Let's try to find practices that prescribe an unusually high amount of opioids. The opioids we'll look for are given in the list below.
In [29]:
opioids = ['morphine', 'oxycodone', 'methadone', 'fentanyl', 'pethidine', 'buprenorphine', 'propoxyphene', 'codeine']

These are generic names for drugs, not brand names. Generic drug names can be found using the 'bnf_code' field in scripts along with the chem table.. Use the list of opioids provided above along with these fields to make a new field in the scripts data that flags whether the row corresponds with a opioid prescription.
In [30]:
f = pd.merge(scripts,chem,left_on=['bnf_code'],right_on=['CHEM SUB'],how='left')
f
Out[30]:

practice
bnf_code
bnf_name
items
nic
act_cost
quantity
CHEM SUB
NAME
0
N85639
0106020C0
Bisacodyl_Tab E/C 5mg
1
0.39
0.47
12
0106020C0
Bisacodyl
1
N85639
0106040M0
Movicol Plain_Paed Pdr Sach 6.9g
1
4.38
4.07
30
0106040M0
Macrogol 3350
2
N85639
0301011R0
Salbutamol_Inha 100mcg (200 D) CFF
1
1.50
1.40
1
0301011R0
Salbutamol
3
N85639
0304010G0
Chlorphenamine Mal_Oral Soln 2mg/5ml
1
2.62
2.44
150
0304010G0
Chlorphenamine Maleate
4
N85639
0401020K0
Diazepam_Tab 2mg
1
0.16
0.26
6
0401020K0
Diazepam
5
N85639
0406000T0
Prochlpzine Mal_Tab 5mg
1
0.97
0.91
28
0406000T0
Prochlorperazine Maleate
6
N85639
0407010F0
Co-Codamol_Cap 30mg/500mg
1
0.84
0.89
24
0407010F0
Co-Codamol (Codeine Phos/Paracetamol)
7
N85639
0407010F0
Zapain_Tab 30mg/500mg
1
3.03
2.82
100
0407010F0
Co-Codamol (Codeine Phos/Paracetamol)
8
N85639
0407010H0
Paracet_Oral Susp Paed 120mg/5ml
1
0.62
0.69
100
0407010H0
Paracetamol
9
N85639
0501011P0
Phenoxymethylpenicillin Pot_Tab 250mg
2
5.94
5.72
160
0501011P0
Phenoxymethylpenicillin (Penicillin V)
10
N85639
0501012G0
Fluclox Sod_Cap 500mg
1
2.17
2.02
28
0501012G0
Flucloxacillin Sodium
11
N85639
0501012G0
Fluclox Sod_Oral Soln 250mg/5ml
1
26.04
24.12
100
0501012G0
Flucloxacillin Sodium
12
N85639
0501013B0
Amoxicillin_Cap 500mg
3
3.81
3.56
63
0501013B0
Amoxicillin
13
N85639
0501013B0
Amoxicillin_Oral Susp 250mg/5ml
1
2.26
2.10
200
0501013B0
Amoxicillin
14
N85639
0501021L0
Cefalexin_Tab 500mg
1
1.75
1.73
20
0501021L0
Cefalexin
15
N85639
0501030I0
Doxycycline Hyclate_Cap 100mg
3
2.61
2.45
24
0501030I0
Doxycycline Hyclate
16
N85639
0501050B0
Clarithromycin_Oral Susp 125mg/5ml
1
4.06
3.77
70
0501050B0
Clarithromycin
17
N85639
0501080W0
Trimethoprim_Tab 200mg
2
1.64
1.55
20
0501080W0
Trimethoprim
18
N85639
0501130R0
Nitrofurantoin_Cap 100mg M/R
1
4.07
3.88
6
0501130R0
Nitrofurantoin
19
N85639
0603020T0
Prednisolone_Tab 5mg
3
4.22
4.14
136
0603020T0
Prednisolone
20
N85639
0902012H0
Gppe Pdr Sach_Dioralyte Relief S/F
1
2.50
2.33
6
0902012H0
Oral Rehydration Salts
21
N85639
1001010J0
Ibuprofen_Oral Susp 100mg/5ml
1
1.78
1.76
100
1001010J0
Ibuprofen
22
N85639
1103010C0
Chloramphen_Eye Dps 0.5%
2
3.04
2.84
20
1103010C0
Chloramphenicol
23
N85639
1103010C0
Chloramphen_Eye Oint 1%
1
2.27
2.11
4
1103010C0
Chloramphenicol
24
N85639
1203010E0
Difflam_P/Spy 0.15% 30ml
1
4.24
3.94
1
1203010E0
Benzydamine Hydrochloride
25
N85639
1502010J0
Instillagel_Gel 6ml Pfs
1
14.05
13.02
60
1502010J0
Lidocaine Hydrochloride
26
N85639
210102301
Able Spacer
1
4.39
4.08
1
NaN
NaN
27
N85647
0301011R0
Salbutamol_Inha 100mcg (200 D) CFF
1
1.50
1.40
1
0301011R0
Salbutamol
28
N85647
0407010H0
Paracet_Oral Susp Paed 120mg/5ml
1
0.62
0.69
100
0407010H0
Paracetamol
29
N85647
0501013B0
Amoxicillin_Oral Susp 250mg/5ml
1
1.13
1.06
100
0501013B0
Amoxicillin
...
...
...
...
...
...
...
...
...
...
973980
H81615
231534515
Pelican_Release Non-Sting Adh Remover A/
2
38.44
35.62
4
NaN
NaN
973981
H81615
231560015
C D Medical_Peel Easy Adh Remover Spy 50
1
28.40
26.29
4
NaN
NaN
973982
H81615
232502625
AMI_Suportx Ostomy Support Mens Shorts D
1
19.62
18.16
2
NaN
NaN
973983
H81615
232502625
AMI_Suportx Ostomy Support Mens Trunks D
1
39.24
36.33
4
NaN
NaN
973984
H81615
233510037
Coloplast_SenSura Midi Closed Bag S/Cove
1
177.50
164.32
60
NaN
NaN
973985
H81615
234506045
Bullen_NA'Scent Odour Eliminator
1
11.97
11.08
236
NaN
NaN
973986
H81615
234510045
Coloplast_Brava Lubricating Deod 240ml
1
8.46
7.84
1
NaN
NaN
973987
H81615
234533745
Respond_OstoMart OstoZyme M/Odour Neutra
1
16.16
14.96
2
NaN
NaN
973988
H81615
234544045
Shaw_Forest Breeze Deod
1
7.70
7.14
2
NaN
NaN
973989
H81615
236007062
Dansac_Nova 1 Drnbl + E/Fld Ileo Bag Sym
1
176.16
163.09
60
NaN
NaN
973990
H81615
236007063
Dansac_Nova 1 Convex E/Fld Drnbl Ileo Ba
1
245.28
227.07
60
NaN
NaN
973991
H81615
236010063
Coloplast_Sensura Mio Maxi DrnblBag Grey
1
99.63
92.23
30
NaN
NaN
973992
H81615
236056062
Welland_FreeStyle Vie Convex Drnbl Pch M
1
220.75
204.36
50
NaN
NaN
973993
H81615
238009080
CliniMed_LBF Ster No Sting Barrier Film
2
76.14
70.50
3
NaN
NaN
973994
H81615
238020980
H & R_Proshield Plus Skin Prote 115g
9
119.28
110.51
12
NaN
NaN
973995
H81615
238031080
3m Health Care_Cavilon No Sting Barrier
4
141.00
130.54
180
NaN
NaN
973996
H81615
238031080
3m Health Care_Cavilon Durable Barrier C
2
16.24
15.05
2
NaN
NaN
973997
H81615
238033680
Opus_SkinSafe Non Sting Prote Film
1
36.76
34.04
1
NaN
NaN
973998
H81615
238048080
Convatec_Orabase Paste 30g
3
8.56
7.93
4
NaN
NaN
973999
H81615
238501085
Hollister_Adapt Barrier Rings 98mm
2
180.08
166.74
80
NaN
NaN
974000
H81615
238510085
Coloplast_Brava Mouldable Rings 48mm x 2
1
58.39
54.05
30
NaN
NaN
974001
H81615
239401097
Hollister_Conform 2 Flat Barrier +Adh 45
1
98.28
90.98
30
NaN
NaN
974002
H81615
239407096
Dansac_Nova 2 Flng 43mm P/Cut 30mm
1
47.46
43.94
15
NaN
NaN
974003
H81615
239407098
Dansac_NovaLife 2 Open Pouch Maxi Opqe 4
1
212.04
196.30
120
NaN
NaN
974004
H81615
239410093
Coloplast_SenSura Flex Drnbl Bag Maxi Op
2
105.02
97.22
60
NaN
NaN
974005
H81615
239410100
Coloplast_SenSura Mio Flex B/Plt 50mm C/
1
78.92
73.06
20
NaN
NaN
974006
H81615
239410100
Coloplast_SenSura Mio Flex Clsd Bag Mini
1
106.36
98.46
60
NaN
NaN
974007
H81615
239410100
Coloplast_SenSura Mio Flex Clsd Bag Midi
1
159.54
147.69
90
NaN
NaN
974008
H81615
239607096
Dansac_Nova 1 Convex Urost Pouch Clr C/F
2
329.76
305.28
60
NaN
NaN
974009
H81615
239610096
Coloplast_SenSura Mio Urost Bag Maxi Tra
1
182.48
168.93
30
NaN
NaN
974010 rows × 9 columns

Now for each practice calculate the proportion of its prescriptions containing opioids.
Hint: Consider the following list: [0, 1, 1, 0, 0, 0]. What proportion of the entries are 1s? What is the mean value?
In [31]:
check = '|'.join(opioids)
chem["opioids"] = chem.NAME.str.contains(check,case = False)
chem
Out[31]:

CHEM SUB
NAME
opioids
0
0101010A0
Alexitol Sodium
False
1
0101010B0
Almasilate
False
2
0101010C0
Aluminium Hydroxide
False
3
0101010D0
Aluminium Hydroxide With Magnesium
False
4
0101010E0
Hydrotalcite
False
5
0101010F0
Magnesium Carbonate
False
6
0101010G0
Co-Magaldrox(Magnesium/Aluminium Hydrox)
False
7
0101010I0
Magnesium Oxide
False
8
0101010J0
Magnesium Trisilicate
False
9
0101010L0
Aluminium & Magnesium & Act Simeticone
False
10
0101010M0
Magaldrate
False
11
0101010N0
Aluminium & Magnesium & Oxetacaine
False
12
0101010P0
Co-Simalcite (Simeticone/Hydrotalcite)
False
13
0101010Q0
Magnesium Hydroxide
False
14
0101010R0
Simeticone
False
15
0101010S0
Calcium Carbonate & Simeticone
False
16
0101010T0
Magnesium Sulphate
False
17
0101010U0
Sodium Citrate
False
18
010101000
Other Antacid & Simeticone Preps
False
19
0101012A0
Gripe Mixtures
False
20
0101012B0
Sodium Bicarbonate
False
21
010101200
Other Sodium Bicarbonate Preps
False
22
0101021B0
Alginic Acid Compound Preparations
False
23
0101021C0
Calcium Carbonate
False
24
0101021E0
Bismuth Subcarbonate
False
25
010102100
Compound Alginates&Prop Indigestion Prep
False
26
0102000AA
Clebopride
False
27
0102000AB
Hyoscyamine Sulfate
False
28
0102000AC
Atropine Sulfate
False
29
0102000AD
Pinaverium Bromide
False
...
...
...
...
3457
1104020AE
Ciclosporin (Eye Anti Inflam)
False
3458
1404000AN
Meningococcal A + C Vaccine
False
3459
1404000AP
Meningococcal C Vaccine
False
3460
1404000AQ
Meningococcal B Vaccine
False
3461
1404000X0
Meningococcal A + C + W135 + Y Vaccine
False
3462
2003
Wound Management & Other Dressings
False
3463
2015
Skin Adhesive Sterile
False
3464
2113
Catheter Maintenance Products
False
3465
2147
Dev For Fungal Nail Infections
False
3466
0601023AV
Saxagliptin/Dapagliflozin
False
3467
0801050CB
Palbociclib
False
3468
1104020AF
Ascorbic Acid (Eye)
False
3469
1203040AA
Diclofenac Sod
False
3470
0105030B0
Ustekinumab (Crohn's)
False
3471
0302000V0
Fluticasone Fuorate (Inh)
False
3472
1305030D0
Ustekinumab (Psoriasis)
False
3473
0302000W0
Reslizumab
False
3474
0501070AE
Fosfomycin Trometamol
False
3475
0801050CC
Ixazomib
False
3476
0908010AP
Elosulfase Alfa
False
3477
0801050CD
Alectinib
False
3478
0102000AJ
Alverine Citrate/Simeticone
False
3479
0501070AF
Dalbavancin Hydrochloride
False
3480
0601060X0
Glucose & Ketone Blood Testing Reagents
False
3481
1001030AC
Baricitinib
False
3482
0102000AK
Eluxadoline
False
3483
0405010T0
Naltrexone/Bupropion
False
3484
0605010AF
Follitropin Delta
False
3485
0406000AH
Rolapitant
False
3486
0908010AQ
Migalastat
False
3487 rows × 3 columns
In [32]:
check = '|'.join(opioids)
chem["opioids"] = chem.NAME.str.contains(check,case = False)
practices.sort_values(by = ['name'], inplace = True, ascending = True)
practices = practices[~practices.duplicated(["code"])]
merged_df = pd.merge(scripts,practices,how='left',left_on='practice',right_on='code')
new_merge = merged_df.merge(chem, left_on='bnf_code', right_on='CHEM SUB')
In [33]:
opioids_per_practice = new_merge.groupby("name")["opioids"].mean()
overall = new_merge["opioids"].mean()
In [34]:
relative_opioids_per_practice = opioids_per_practice - overall

How do these proportions compare to the overall opioid prescription rate? Subtract off the proportion of all prescriptions that are opioids from each practice's proportion.

Now that we know the difference between each practice's opioid prescription rate and the overall rate, we can identify which practices prescribe opioids at above average or below average rates. However, are the differences from the overall rate important or just random deviations? In other words, are the differences from the overall rate big or small?
To answer this question we have to quantify the difference we would typically expect between a given practice's opioid prescription rate and the overall rate. This quantity is called the standard error, and is related to the standard deviation, 
σ
σ
. The standard error in this case is
σ
n
−
−
√

σn
where 
n
n
is the number of prescriptions each practice made. Calculate the standard error for each practice. Then divide relative_opioids_per_practice by the standard errors. We'll call the final result opioid_scores.
In [35]:
standard_div = new_merge["opioids"].std()
n = new_merge.groupby("name")["opioids"].count()
import numpy as np
standard_error_per_practice = standard_div/np.sqrt(n)
opioid_scores = relative_opioids_per_practice/standard_error_per_practice
h=opioid_scores[:100]
tuple(h)
Out[35]:
(2.042750264263512,
 0.12840935966485323,
 0.9750025099475605,
 0.6792915126217348,
 -0.05754089987679946,
 -0.5436825171433692,
 0.7404526973367067,
 -1.095989836496652,
 -0.3329348094802244,
 -3.42922783954884,
 -0.5047955893869974,
 -0.903792432808443,
 -2.2392082342562896,
 -1.546503427784878,
 -0.02411506274934926,
 0.37112494452074757,
 -1.2986988302623197,
 -1.9565569472002897,
 0.6224498610637532,
 -3.114929294881899,
 2.2857408229912974,
 -0.6073082028884207,
 0.7673369909941493,
 0.20405745755741816,
 -1.2016538575504492,
 -0.042011502984152384,
 0.7036045556020236,
 1.479560978327441,
 1.2444409593122703,
 -1.6680767067157964,
 -0.33641149729897346,
 2.0347515962771467,
 -0.019605298931671476,
 0.5261442111743129,
 -0.9181327396240229,
 1.932526336114186,
 -2.827683498324077,
 -1.4285784870014027,
 0.4403010392899367,
 -2.9368779353840146,
 0.26044643596976974,
 0.20732563319970032,
 0.8703872897985474,
 3.285466488509827,
 2.558510806138602,
 3.2086061856768966,
 -1.7607648364519195,
 -1.7414950267318219,
 0.500155102842041,
 1.3677535829369982,
 3.3809912598809437,
 1.6972137189303296,
 0.14399658860622333,
 -2.762960617223706,
 1.1917229742573863,
 -0.2758184127336634,
 -0.20051140775133183,
 0.7104246393497252,
 0.1307567040478587,
 0.9109839865314264,
 1.231275503014394,
 -1.4860779772809949,
 -1.6609442272929595,
 -0.7907342565142352,
 0.7588021824067931,
 -0.31092539427401017,
 0.8364064569840218,
 -0.7177797160950576,
 -1.9383297159493953,
 1.538626537354757,
 0.14091001407299322,
 -1.7368443979375585,
 1.4570234297244284,
 -0.972044354739357,
 -2.295420833721535,
 -1.4660983565343466,
 -1.3680330220746235,
 -1.2045883559753503,
 -3.007696136441691,
 -0.5607877474015218,
 0.49760048319241856,
 1.722794635825227,
 4.988317952893894,
 -1.15253582371049,
 1.7442913714335704,
 0.33984211798012337,
 -0.09839579521405967,
 0.7410649139622233,
 1.8100415777696632,
 -2.12699833015426,
 -0.41375146875934954,
 0.8623195965927544,
 0.7653941918719358,
 2.847813000987497,
 4.207365476772379,
 -0.36908176104368723,
 -0.5452415534195163,
 1.925520296849967,
 -0.031117062029002423,
 -0.2655074370948234)

The quantity we have calculated in opioid_scores is called a z-score:
X
¯
−μ
σ
2
/n
−
−
−
−
√

X¯−μσ2/n
Here 
X
¯
X¯
corresponds with the proportion for each practice, 
μ
μ
corresponds with the proportion across all practices, 
σ
2
σ2
corresponds with the variance of the proportion across all practices, and 
n
n
is the number of prescriptions made by each practice. Notice 
X
¯
X¯
and 
n
n
will be different for each practice, while 
μ
μ
and 
σ
σ
are determined across all prescriptions, and so are the same for every z-score. The z-score is a useful statistical tool used for hypothesis testing, finding outliers, and comparing data about different types of objects or events.
Now that we've calculated this statistic, take the 100 practices with the largest z-score. Return your result as a list of tuples in the form (practice_name, z-score, number_of_scripts). Sort your tuples by z-score in descending order. Note that some practice codes will correspond with multiple names. In this case, use the first match when sorting names alphabetically.
In [36]:
len(scripts)
Out[36]:
973193
In [37]:
chem['NAME'] = chem['NAME'].str.lower()
opioids = ['morphine', 'oxycodone', 'methadone', 'fentanyl', 'pethidine', 'buprenorphine', 'propoxyphene', 'codeine']
chem['is_opioid'] = chem['NAME'].str.contains('|'.join(opioids))*1
In [38]:
chem_op = chem
chem_op['NAME'] = chem_op['NAME'].str.lower()
chem_op['is_op'] = chem_op.NAME.str.contains("|".join(opioids)) * 1
chem_op['is_op'] = pd.to_numeric(chem_op['is_op'])
chem_op = chem_op[chem_op['is_op'] == 1]
chem_op_list = chem_op['CHEM SUB'].values
chem_op_list
Out[38]:
array(['0104020D0', '0104020N0', '0309010C0', '0309010N0', '0309010S0',
       '0309020AC', '0407010F0', '0407010M0', '0407010N0', '0407010R0',
       '0407010T0', '0407010V0', '0407020AD', '0407020AE', '0407020AF',
       '0407020A0', '0407020B0', '0407020C0', '0407020E0', '0407020G0',
       '0407020K0', '0407020M0', '0407020N0', '0407020P0', '0407020Q0',
       '0407020V0', '0407020Z0', '040702010', '040702020', '0409010A0',
       '0410030A0', '0410030C0', '0704050G0', '1501043F0', '1502010Z0'],
      dtype=object)
In [39]:
len(chem_op)
Out[39]:
35
In [40]:
chem_op['is_op'].mean()
Out[40]:
1.0
In [41]:
chem_op['is_op'].mean()
Out[41]:
1.0
In [42]:
scripts[scripts['practice'] == '8B09S3']
Out[42]:

practice
bnf_code
bnf_name
items
nic
act_cost
quantity

In [43]:
def is_op_check(v):
    if v in chem_op_list:
        return 1
    else:
        return 0          
In [44]:
scripts_m = scripts
scripts_m['is_op'] = scripts_m['bnf_code'].apply(is_op_check)
m_df_scripts_chem = scripts_m
m_df_op_prop['practice_count'].sum()
m_df_op_prop['op_prop'].mean()
m_df_op_prop['op_prop'].mean()
m_df_op_prop['op_prop'] = m_df_op_prop['is_op'] /  m_df_op_prop['practice_count']

---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-44-9ca8e5ad1505> in <module>()
      2 scripts_m['is_op'] = scripts_m['bnf_code'].apply(is_op_check)
      3 m_df_scripts_chem = scripts_m
----> 4 m_df_op_prop['practice_count'].sum()
      5 m_df_op_prop['op_prop'].mean()
      6 m_df_op_prop['op_prop'].mean()

NameError: name 'm_df_op_prop' is not defined
In [ ]:
x_bar=scripts_m['is_op'].mean()
x_bar
In [ ]:
sigma = scripts['is_op'].std()
sigma
In [ ]:
mu = scripts['is_op'].mean()
mu
In [ ]:
scripts['is_op'].sum()
In [ ]:
mu/np.sqrt((sigma)**2/100)
In [ ]:
n_value=len(scripts['is_op'])
n_value
In [ ]:
import re
def ismatch(name):
    for opioid in opioids:
        m=re.search( opioid.upper(), name.upper())
        if m:
            return True
    return False
In [ ]:
import pandas as pd
anomalies = pd.merge(scripts,chem,left_on='bnf_code',right_on='CHEM SUB',how='inner')
In [ ]:
 check = '|'.join(opioids)
 chem_df['is_opioid'] = chem_df.NAME.str.contains(pat=check,case=False) 
 chem_df['is_opioid'] =chem_df['is_opioid'] * 1 
 flag =chem_df['is_opioid'] ==True 
 test_case = chem_df[flag] 
 chem_sub_list = list(test_case['CHEM SUB'])
In [ ]:
chem = chem[~chem.duplicated(["CHEM SUB"])]
chem['opioids'] = chem.NAME.str.contains('|'.join(opioids),case = False)*1 

# merged scripts and chem, filled NAs with 0

sc_merged = scripts.merge(chem, how = 'left', left_on='bnf_code', right_on='CHEM SUB')
sc_merged = sc_merged.fillna(0)

# sort practices and removed duplicates

prac_sort = practices.sort_values(by=['name'], ascending=True)
prac_sort = prac_sort[~prac_sort.duplicated(["code"])]

# joined practice with the previous merge

spc_joined = pd.merge(sc_merged,prac_sort,how='inner',left_on='practice',right_on='code')
spc_joined = spc_joined.fillna(0)

# calculated mean() and std()

spc_mean = spc_joined.opioids.mean()
spc_std =spc_joined.opioids.std()
print(spc_mean,  spc_std)

#calculated proportion

opioids_per_practice = spc_joined.groupby("practice")["opioids"].mean()

# represent as a dataframe

spc_final = pd.DataFrame(opioids_per_practice)
spc_final.reset_index(inplace=True)

#relative practice column

spc_final['relative_opioids'] = spc_final.opioids- spc_mean

# calculated n
n = spc_joined.groupby("practice")["opioids"].count()

# inscribed n to the dataframe

sc = pd.DataFrame(n)
sc.reset_index(inplace=True)
spc_final   = pd.merge(spc_final, sc, how="left", on="practice")


# calculate std error

spc_final['std_error'] = spc_std/spc_final.opioids_y.apply(np.sqrt)

# calculate z_scores

spc_final['opioid_scores'] = spc_final.relative_opioids/spc_final.std_error

# merge with practices to get the other names
result =pd.merge(spc_final.sort_values (by=["opioid_scores"], ascending=False), prac_sort, how="left", left_on="practice", right_on="code")[["name", "opioid_scores", "opioids_y"]]
result.head()

# calculate the anomalies

m_anomalies = [tuple(x) for x in result.values]

anomalies =m_anomalies[:100]
anomalies[:5]

/opt/conda/lib/python3.6/site-packages/ipykernel_launcher.py:2: SettingWithCopyWarning: 
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
  
In [ ]:
grader.score.dw__script_anomalies(anomalies)

Question 5: script_growth
Another way to identify anomalies is by comparing current data to historical data. In the case of identifying sites of drug abuse, we might compare a practice's current rate of opioid prescription to their rate 5 or 10 years ago. Unless the nature of the practice has changed, the profile of drugs they prescribe should be relatively stable. We might also want to identify trends through time for business reasons, identifying drugs that are gaining market share. That's what we'll do in this question.
We'll load in beneficiary data from 6 months earlier, June 2016, and calculate the percent growth in prescription rate from June 2016 to January 2017 for each bnf_name. Normalize the percent growth in prescriptions of individual items by the percent change in total number of prescriptions (think about whether this normalization should be a division or a subtraction). We'll return the 50 items with largest growth and the 50 items with the largest shrinkage (i.e. negative percent growth) as a list of tuples sorted by growth rate in descending order in the format (script_name, growth_rate, raw_2016_count). You'll notice that many of the 50 fastest growing items have low counts of prescriptions in 2016. Filter out any items that were prescribed less than 50 times.
In [11]:
# load data
scripts16 = pd.read_csv('./dw-data/201606scripts_sample.csv.gz', compression='gzip')
scripts17 = pd.read_csv('./dw-data/201701scripts_sample.csv.gz', compression='gzip')

# format date times and combine 16 and 17 script data
scripts16['year'] = str(2016)
scripts17['year'] = str(2017)

scripts16['year'] = pd.to_datetime(scripts16['year'])
scripts17['year'] = pd.to_datetime(scripts17['year'])

scripts_yrs = [scripts16, scripts17]

df1 = pd.concat(scripts_yrs)

# extract necessary columns
df1 = df1[['bnf_name','items','year']]

# set index
df1 = df1.set_index(['year','bnf_name'])
In [16]:
import pandas as pd
scripts16_2=scripts16[['bnf_name','items']]
scripts16_2.reset_index(inplace=True)
scripts16_2=pd.DataFrame(scripts16_2.groupby('bnf_name').count())
scripts16_2.reset_index(inplace=True)

scripts_2=scripts[['bnf_name','items']]
scripts_2.reset_index(inplace=True)
scripts_2=pd.DataFrame(scripts_2.groupby('bnf_name').count())
scripts_2.reset_index(inplace=True)

scripts16_4=scripts16_2.merge(scripts_2,on='bnf_name', how='left')
scripts16_4.reset_index(inplace=True)
scripts16_4['items_y'].fillna(0, inplace=True)

scripts16_4['rate']=(scripts16_4['items_y']-scripts16_4['items_x'])/scripts16_4['items_x']
scripts16_4=scripts16_4[scripts16_4['items_x']>50]

growth_rate=scripts16_4[['bnf_name','rate','items_x',]].sort_values(by=['rate'],ascending=False).reset_index(drop=True)
top_50=growth_rate.head(50)
bottom_50=growth_rate.tail(50)
script_growth=[tuple(i) for i in pd.concat([top_50, bottom_50]).values]
In [17]:
grader.score.dw__script_growth(script_growth)

==================
Your score:  0.6600000000000004
==================

Question 6: rare_scripts
Does a practice's prescription costs originate from routine care or from reliance on rarely prescribed treatments? Commonplace treatments can carry lower costs than rare treatments because of efficiencies in large-scale production. While some specialist practices can't help but avoid prescribing rare medicines because there are no alternatives, some practices may be prescribing a unnecessary amount of brand-name products when generics are available. Let's identify practices whose costs disproportionately originate from rarely prescribed items.
First we have to identify which 'bnf_code' are rare. To do this, find the probability 
p
p
of a prescription having a particular 'bnf_code' if the 'bnf_code' was randomly chosen from the unique options in the beneficiary data. We will call a 'bnf_code' rare if it is prescribed at a rate less than 
0.1p
0.1p
.

Finally compute the z-scores. Return the practices with the top 100 z-scores in the form (post_code, practice_name, z-score). Note that some practice codes will correspond with multiple names. In this case, use the first match when sorting names alphabetically.
In [15]:
import math
import gzip
import numpy as np
import pandas as pd
from static_grader import grader

with gzip.open ( './dw-data/201701scripts_sample.csv.gz', 'rb' ) as f:
    scripts_all_columns = pd.read_csv ( f )

with gzip.open ( './dw-data/practices.csv.gz', 'rb' ) as f:
    practices = pd.read_csv ( f )

practices.columns = [ 'practice', 'name', 'addr_1', 'addr_2', 'borough', 'village', 'post_code']
practices = practices[ [ 'practice', 'name' ] ].sort_values ( by = [ 'name' ], ascending = True)
practices = practices [ ~practices.duplicated ( [ 'practice' ] ) ]

scripts = scripts_all_columns [ [ 'practice', 'bnf_code', 'act_cost' ] ] [ : ]
total_script_count = len ( scripts )

scripts [ 'bnf_count' ] = 1
unique_bnf = scripts.groupby ( 'bnf_code', as_index = False ) [ 'bnf_count' ].count()
rare_threshold = 0.1 / len ( unique_bnf )

rare_cond = unique_bnf [ 'bnf_count' ] / total_script_count < rare_threshold

rare_bnf = unique_bnf [ rare_cond ] [ 'bnf_code' ]

practice_rare_cost = scripts [ scripts ['bnf_code'].isin ( rare_bnf ) ]\
                            .groupby ( 'practice', as_index = False ).agg ( { 'act_cost': 'sum' } )

practice_all_cost = scripts.groupby ( 'practice', as_index = False ).agg ( { 'act_cost': 'sum' } )

rare_cost_prop =  practice_all_cost.merge ( practice_rare_cost, on = 'practice', suffixes = [ '_all_bnf', '_rare_bnf' ] )
# rare_cost_prop = rare_cost_prop [ ~rare_cost_prop.duplicated ( [ 'practice' ] ) ]

rare_cost_prop [ 'rare_prop' ] = rare_cost_prop.apply ( lambda df: df.act_cost_rare_bnf / df.act_cost_all_bnf, axis = 1 )
# print(rare_cost_prop.head(100))

overall_rare_cost_prop = rare_cost_prop.act_cost_rare_bnf.sum() / rare_cost_prop.act_cost_all_bnf.sum()

rare_cost_prop = rare_cost_prop [ [ 'practice', 'rare_prop' ] ] [ : ]

rare_cost_prop [ 'relative' ] = rare_cost_prop.rare_prop - overall_rare_cost_prop
# print(rare_cost_prop)

std_error = rare_cost_prop.relative.std ()
# print(std_error)#0.06353378513687823

rare_cost_prop [ 'z_score' ] = rare_cost_prop.relative / std_error

rare_scores = rare_cost_prop.merge ( practices, on = 'practice' ) [ [ 'practice', 'name', 'z_score' ] ] [ : ]
rare_scores.sort_values ( by = 'z_score', ascending = False, inplace = True )
# print(rare_scores.head(100))

rare_scripts = [ ( k [ 1 ], k [ 2 ], k [ 3 ] ) for k in rare_scores.itertuples() ] [ : 100 ]
# rare_scripts
grader.score.dw__rare_scripts(rare_scripts)

==================
Your score:  1.0
==================
