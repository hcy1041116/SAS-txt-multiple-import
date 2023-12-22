# SAS-txt-multiple-import
This project can import multiple txt files into SAS, using SAS and python.

First, we capture file names of the targeted folder.
And then import files by PROC IMPORT, if all files successfully import then the works are done.
But if have any problems, we used the log.txt to adjust SAS provided data infile code.
The log.txt is messy, so we clean it using python.
***SAS***
```
filename indata pipe 'dir D:\C678/b';
data file_list;
 length fname $32;
 infile indata truncover; /* infile statement for file names */
 input fname $32.; /* read the file names from the directory */
position = find(fname, '.');
extension = substr(fname, position+1, 3);
if extension='txt';
nposition=find(fname,'C678_');
nfind=substr(fname, nposition+5, 2);
if nfind in ('11', '12', '13', '14', '15');
run; 
data txt_list;
set  file_list;;
 call symput ('num_files',_n_); /* store the record number in a macro variable */
run;


*%put &num_files;
%macro fileread;
%do j=1 %to &num_files;
proc printto log = 'D:\C678\log.txt';
data _null_;
set txt_list;
if _n_=&j.;
call symput ('filein',fname);
tempfile = compress(fname, '.txt');
call symput('tempfile', tempfile);
run;
%put &filein;
PROC IMPORT OUT= WORK.&tempfile
            DATAFILE= "D:\C678\&filein" 
            DBMS=TAB REPLACE;
     GETNAMES=YES;
     DATAROW=2; 
RUN;
proc printto; run;
%end; /* end of do-loop with index j */
%mend fileread;
%fileread; 




PROC IMPORT OUT= WORK.test1
            DATAFILE= "D:\C678\&fname" 
            DBMS=TAB REPLACE;
     GETNAMES=YES;
     DATAROW=2; 
RUN;
```


***PYTHON***
```
import re

# 打開並讀取.txt檔案，指定使用 'latin-1' 編碼方式
with open('log.txt', 'r', encoding='utf-8') as file:
    program_code = file.read()
pattern = re.compile(r'(data\s+work\.\s*[\s\S]*?RUN;)', re.IGNORECASE)
matches = pattern.findall(program_code)

'''
# 印出結果（去除每行前面的數字）
for match in matches:
    match_lines = match.split('\n')
    formatted_match = '\n'.join([line.split(None, 1)[1] if len(line.split(None, 1)) > 1 else line for line in match_lines])
    print('PROC '+formatted_match)
    print()
    '''

# 將結果保存到新的txt檔案中
output_file_path = 'output.txt'
with open(output_file_path, 'w') as output_file:
    # 寫入結果（去除每行前面的數字）
    for match in matches:
        match_lines = match.split('\n')
        formatted_match = '\n'.join([line.split(None, 1)[1] if len(line.split(None, 1)) > 1 else line for line in match_lines])
        result = 'DATA '+formatted_match+'\n'
        output_file.write(result+'\n')
```
