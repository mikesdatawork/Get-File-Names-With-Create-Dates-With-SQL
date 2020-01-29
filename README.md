![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Get File Names With Create Dates With SQL
**Post Date: February 5, 2018**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some quick logic that will return both the Create Date, and File Name of your files in the designated folder. In this case I'm looking in forder C:\SQLIMPORTS. This was originally created to use in an automated ETL process. Could use alittle optimization, but works great.</p>      


## SQL-Logic
```SQL
-- get simple file list
declare @path       varchar(255) = 'C:\SQLIMPORTS\'
declare @files      table ([subdirectory] varchar(255), [depth] int, [file] int)
insert into     @files exec master..xp_dirtree @path, 1, 1; delete from @files where [file] = 0 
 
if object_id('tempdb..#import_folder') is not null
drop table      #import_folder
create table        #import_folder  ([file_info] nvarchar(255))
declare         @file_list  table ([file_name] nvarchar(255))
 
-- get file meta data
Insert into     #import_folder exec xp_cmdshell 'dir C:\SQLIMPORTS\*.*'
delete from     #import_folder where [file_info] is null 
delete from     #import_folder where [file_info] like '%dir%' 
delete from     #import_folder where [file_info] like '%volume%' 
delete from     #import_folder where [file_info] like '%bytes%'
 
-- combine meta data with file name
if object_id('tempdb..#files_found') is not null
drop table      #files_found
create table        #files_found ([create_date] datetime, [file_name] nvarchar(255))
 
declare @compile_list       varchar(max)
set @compile_list       = ''
select  @compile_list       = @compile_list + 
'insert into #files_found  ([create_date], [file_name]) select cast(left([file_info], 20) as datetime), ''' + [subdirectory] + ''' from #import_folder where [file_info] like ''%' + [subdirectory] + '%'';' + char(10)
from    @files
exec    (@compile_list)
 
select
    [create_date]
,   [file_name]
,   'file_type'     = upper(right([file_name], 3)) 
from
    #files_found order by [create_date] desc
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

 
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

