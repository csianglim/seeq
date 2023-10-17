# Seeq
A cheatsheet for Seeq

# How to install Latex in SDL without sudo

Reference: https://www.tug.org/texlive/quickinstall.html

```
1.  mkdir /home/datalab/tex && mkdir tmp && cd tmp && wget https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz && zcat install-tl-unx.tar.gz | tar xf - # working directory of your choice
2.  cd install-tl-*
3.  perl ./install-tl --scheme=basic --texdir=/home/datalab/tex # make the folder first
```
Navigate to `/home/datalab/tex` then install these packages:

```https://stackoverflow.com/questions/55746749/latex-equations-do-not-render-in-google-colaboratory-when-using-matplotlib
1. cd /home/datalab/tex/bin/x86_64-linux && ./tlmgr install collection-latexrecommended collection-fontsrecommended collection-fontsextra ec dvipng
```

Install `type1cm` manually for matplotlib to work. [Reference](https://stackoverflow.com/questions/55746749/latex-equations-do-not-render-in-google-colaboratory-when-using-matplotlib):

```
1. wget http://mirrors.ctan.org/macros/latex/contrib/type1cm.zip && unzip type1cm.zip -d /tmp/type1cm && mkdir ~/tex/texmf-dist/tex/latex/type1cm
2. ~/tex/bin/x86_64-linux/latex /tmp/type1cm/type1cm/type1cm.ins
3. cp /tmp/type1cm/type1cm/type1cm.sty ~/tex/texmf-dist/tex/latex/type1cm
4. ~/tex/bin/x86_64-linux/texhash
```
Finally, add tex to PATH in the notebook
```
import os
os.environ["PATH"] += os.pathsep + "/home/datalab/tex/bin/x86_64-linux" # for latex
print(os.environ['PATH'])
```

# Capsule Properties

```
// Lab timer indicator condition
$a = $clt.replace('On','1').toCondition().keep('Value', isEqualTo('1')).removeLongerThan(1month)

// Lab load switch condition
$b = $dlls.replace('On','1').toCondition().keep('Value', isEqualTo('1')).removeLongerThan(1month)

// Sampled and loaded condition
$c = $a.touches($b.beforestart(1min))

// grab the lab and inferential predictions and calculate deltas as properties
$d = $c
.transform($capsule -> $capsule.setProperty('Uncorrected Inferential', $au.toScalars($capsule).first()))
.transform($capsule -> $capsule.setProperty('Corrected Inferential', $a2c.toScalars($capsule).first()))
.transform($capsule -> $capsule.setProperty('Lab', $blr.toScalars($capsule).last()))
.transform($capsule -> $capsule.setProperty('Delta - Uncorrected',
           $au.toScalars($capsule).first()-$blr.toScalars($capsule).last()))
.transform($capsule -> $capsule.setProperty('Delta - Corrected',
           $a2c.toScalars($capsule).first()-$blr.toScalars($capsule).last()))
           
return $d
```

# Bulk load tags in Seeq Data Lab
How to bulk load tags from SDL back into the workbench or export as CSV. `spy.push()` only displays 10 signals by default. See [https://www.seeq.org/index.php?/forums/topic/1105-increase-the-number-of-displayed-signals-from-10-when-spypush-is-used/](https://www.seeq.org/index.php?/forums/topic/1105-increase-the-number-of-displayed-signals-from-10-when-spypush-is-used/) for a workaround using `display_items`:

```python
from seeq import spy
import pandas as pd
my_items = pd.DataFrame({
    'Name': [
        'tag1',
        'tag2',
        'tag3'
    ],
    'Datasource Name': 'PI_SERVER_NAME'
})

items = spy.search(my_items)
for i in my_items['Name']:
    if i not in items['Name'].tolist():
        print('TAG NOT FOUND: ', i)
        
df = spy.pull(items,
         start='2022-01-24T02:00:00',
         end='2022-01-26T14:00:00',
         grid='1min')

df.index = df.index.strftime("%m/%d/%Y %I:%M %p")    

arrs = [items['Description'].values.tolist(), df.columns.tolist()]
header = pd.MultiIndex.from_arrays(arrs,
                                   names=['Description','Tag Name'])
df2 = df
df2.columns = header
df2.to_csv('df.csv')

# Push back to workbench
push_results = spy.push(metadata=items)
url = 'XXXX' # Pull the workbook. Use the URL of the Workbook where the signals where pushed 4# You can take it from the output of the above command
wb = spy.workbooks.pull(url)[0]
ws = wb.worksheets[0]
ws.display_items = push_results # add all the pushed items to the worksheet

# push the workbook with the modified worksheet back to Seeq
spy.workbooks.push(wb)
```
