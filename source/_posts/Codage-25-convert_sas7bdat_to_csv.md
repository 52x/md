Convert Large Size `.sas7bdat` (SAS Database file) to `.csv`

R受限于内存，不可能读取太大的文件，现有的读取SAS文件的包(如`haven`, `sas7bdat`, `foreign`等)，并不提供类似`fread()`函数的参数功能，用于选取指定的列、跳过特定的行数以及读取指定的行数。

<!-- more -->

所以能想到的方案是先将`.sas7bdat`文件转换成`.csv`，然后用`fread`循环读取转换后的文件中的不同行数。(不要问我为什么不能直接从SAS中导出`.csv`文件...)




http://stackoverflow.com/questions/22213203/how-to-read-in-large-sas7bdat-dataset-in-r
http://www.oview.co.uk/dsread/
http://benpiper.com/2011/11/forcing-apps-to-run-in-32-bit-mode-in-a-64-bit-windows-environment/