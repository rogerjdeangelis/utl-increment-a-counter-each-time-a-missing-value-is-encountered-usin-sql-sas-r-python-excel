    %let pgm=utl-increment-a-counter-each-time-a-missing-value-is-encountered-usin-sql-sas-r-python-excel;

    %sto_submission;

    Increment a counter each time a missing value is encountered usin sql sas r python excel

    CONTENTS
       1 sas sql
       2 r sql      exactly the same code as sas (see after problem)
       3 python sql
       4 excel sql
       5 sas datastep
         Keintz, Mark
         mkeintz@outlook.com

         SOAPBOX ON

         Mark uses a very interesting sas statement.
         The lhs of the sas statement below is an expression.

         The code increments a counter each time the censor is missing

         cnt+first.censor and censor^=.;

         This is like

         name="ROGER_XX";
         substr(name,7,2)="02";

         output

         ROGER_02

         SOAPBOX OFF

    github
    https://tinyurl.com/mt7k3k4e
    https://github.com/rogerjdeangelis/utl-increment-a-counter-each-time-a-missing-value-is-encountered-usin-sql-sas-r-python-excel

    communities.sas
    https://tinyurl.com/z9yr9kef
    https://communities.sas.com/t5/SAS-Programming/How-to-create-an-incremental-grouping-variable-based-on-lagged/m-p/812734#M320682

    SOAPBOX ON
       This is an interesting sql structure, nested selecct clauses without a join.
       The inner select, counts the NAs that are less than or equal the current outer rownum.
    SOAPBOX ON

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    /**************************************************************************************************************************/
    /*       INPUT            |                       PROCESS(Self expanatory SQL)                 |    OUTPUT                */
    /*       =====            |                       ============================                 |    ======                */
    /* ROWNUM  CENSOR         |TEMP                                                                |ROWNUM CENSOR NA_COUNTER  */
    /*     1     1            | 0  count inner rownums(1)        <= outer rownum(1) and censor = NA|    1    1          1     */
    /*     2    NA            | 1  count inner rownums(1,2)      <= outer rownum(2) and censor = NA|    2   NA         NA     */
    /*     3     1            | 1  count inner rownums(1,2,3)    <= outer rownum(3) and censor = NA|    3    1          2     */
    /*     4     1            | 1                                                                  |    4    1          2     */
    /*     5    NA            | 2  count inner rownums(1,2,,4,5) <= outer rownum(5) and censor = NA|    5   NA         NA     */
    /*     6     1            | 2                                                                  |    6    1          3     */
    /*     7     1            | 2                                                                  |    7    1          3     */
    /*     8     1            | 2                                                                  |    8    1          3     */
    /*     9     1            | 2                                                                  |    9    1          3     */
    /*    10    NA            | 3                                                                  |   10   NA         NA     *
    /*    11     1            | 3                                                                  |   11    1          4     */
    /*    12    NA            | 4                                                                  |   12   NA         NA     */
    /*    13    NA            | 4                                                                  |   13   NA         NA     */
    /*                        |---------------------------------------------------------------------------------------------- */
    /* options                |                                                                    |                          */
    /* validvarname=upcase;   |  1 SAS SQL  (exactly same code in r python excel - see below)      | ROWNUM CENSOR NA_COUNTER */
    /* libname sd1 "d:/sd1";  |  =========                                                         |                          */
    /* data sd1.have;         |                                                                    |    1      1        1     */
    /*  input rownum censor;  |  proc sql;                                                         |    2      .        .     */
    /* cards4;                |   create                                                           |    3      1        2     */
    /* 01 1                   |       table want as                                                |    4      1        2     */
    /* 02 .                   |   /*---- outer select ----*/                                       |    5      .        .     */
    /* 03 1                   |   select                                                           |    6      1        3     */
    /* 04 1                   |      otr.rownum                                                    |    7      1        3     */
    /* 05 .                   |     ,otr.censor                                                    |    8      1        3     */
    /* 06 1                   |     ,( /*---- inner select ----*/                                  |    9      1        3     */
    /* 07 1                   |        select                                                      |   10      .        .     */
    /* 08 1                   |           count(inr.rownum)                                        |   11      1        4     */
    /* 09 1                   |        from                                                        |   12      .        .     */
    /* 10 .                   |          sd1.have as inr                                           |   13      .        .     */
    /* 11 1                   |        where                                                       |                          */
    /* 12 .                   |          inr.rownum <= otr.rownum and                              |                          */
    /* 13 .                   |          inr.censor is null                                        |                          */
    /* ;;;;                   |      )+1*censor as na_counter                                      |                          */
    /* run;quit;              |   from                                                             |                          */
    /*                        |       sd1.have as otr                                              |                          */
    /*                        |   order                                                            |                          */
    /*                        |       by otr.rownum                                                |                          */
    /*                        |  ;quit;                                                            |                          */
    /*                        |                                                                    |                          */
    /*------------------------------------------------------------------------------------------------------------------------*/
    /*                        |   5 SAS DATASTEP                                                   |     Censor    NA         */
    /*                        |   =========                                                        |        1       1         */
    /*                        |                                                                    |        .       .         */
    /*                        |   data want (drop=_:);                                             |        1       2         */
    /*                        |     set sd1.have;                                                  |        1       2         */
    /*                        |     by censor notsorted;                                           |        .       .         */
    /*                        |     _na+first.censor and censor^=.;                                |        1       3         */
    /*                        |     if censor^=. then na=_na;                                      |        1       3         */
    /*                        |   run;                                                             |        1       3         */
    /*                        |                                                                    |        1       3         */
    /*                        |                                                                    |        .       .         */
    /*                        |                                                                    |        1       4         */
    /*                        |                                                                    |        .       .         */
    /*                        |                                                                    |        .       .         */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options
    validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input rownum censor;
    cards4;
    01 1
    02 .
    03 1
    04 1
    05 .
    06 1
    07 1
    08 1
    09 1
    10 .
    11 1
    12 .
    13 .
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /* SD1.HAVE total obs=13                                                                                                  */
    /* Obs    ROWNUM    CENSOR                                                                                                */
    /*                                                                                                                        */
    /*   1       1         1                                                                                                  */
    /*   2       2         .                                                                                                  */
    /*   3       3         1                                                                                                  */
    /*   4       4         1                                                                                                  */
    /*   5       5         .                                                                                                  */
    /*   6       6         1                                                                                                  */
    /*   7       7         1                                                                                                  */
    /*   8       8         1                                                                                                  */
    /*   9       9         1                                                                                                  */
    /*  10      10         .                                                                                                  */
    /*  11      11         1                                                                                                  */
    /*  12      12         .                                                                                                  */
    /*  13      13         .                                                                                                  */
    /**************************************************************************************************************************/

    /*                             _
    / |  ___  __ _ ___   ___  __ _| |
    | | / __|/ _` / __| / __|/ _` | |
    | | \__ \ (_| \__ \ \__ \ (_| | |
    |_| |___/\__,_|___/ |___/\__, |_|
                                |_|
    */

    proc sql;
     create
         table want as
     /*---- outer select ----*/
     select
        otr.rownum
       ,otr.censor
       ,( /*---- inner select ----*/
          select
             count(inr.rownum)
          from
            sd1.have as inr
          where
            inr.rownum <= otr.rownum and
            inr.censor is null
        )+1*censor as na_counter
     from
         sd1.have as otr
     order
         by otr.rownum
    ;quit;

    /**************************************************************************************************************************/
    /* WRK.WANT total obs=13                                                                                                  */
    /* Obs    ROWNUM    CENSOR    NA_COUNTER                                                                                  */
    /*                                                                                                                        */
    /*   1       1         1           1                                                                                      */
    /*   2       2         .           .                                                                                      */
    /*   3       3         1           2                                                                                      */
    /*   4       4         1           2                                                                                      */
    /*   5       5         .           .                                                                                      */
    /*   6       6         1           3                                                                                      */
    /*   7       7         1           3                                                                                      */
    /*   8       8         1           3                                                                                      */
    /*   9       9         1           3                                                                                      */
    /*  10      10         .           .                                                                                      */
    /*  11      11         1           4                                                                                      */
    /*  12      12         .           .                                                                                      */
    /*  13      13         .           .                                                                                      */
    /**************************************************************************************************************************/

    /*___                     _
    |___ \   _ __   ___  __ _| |
      __) | | `__| / __|/ _` | |
     / __/  | |    \__ \ (_| | |
    |_____| |_|    |___/\__, |_|
                           |_|
    */

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    options(sqldf.dll = "d:/dll/sqlean.dll")
    have<-read_sas("d:/sd1/have.sas7bdat")
    print(have)
    want <- sqldf("
      select
         otr.rownum
        ,otr.censor
        ,(select count(inr.rownum)
      from
         have as inr
      where
         inr.rownum <= otr.r ownum and
         inr.censor is null)*censor+1 as na_counter
      from
          have as otr
      order
          by otr.rownum
    ")
    want
    fn_tosas9x(
          inp    = want

         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /* R                            | SAS                                                                                     */
    /*    ROWNUM CENSOR NA_COUNTER  | ROWNAMES    ROWNUM    CENSOR    NA_COUNTER                                              */
    /*                              |                                                                                         */
    /* 1       1      1          1  |     1          1         1           1                                                  */
    /* 2       2     NA         NA  |     2          2         .           .                                                  */
    /* 3       3      1          2  |     3          3         1           2                                                  */
    /* 4       4      1          2  |     4          4         1           2                                                  */
    /* 5       5     NA         NA  |     5          5         .           .                                                  */
    /* 6       6      1          3  |     6          6         1           3                                                  */
    /* 7       7      1          3  |     7          7         1           3                                                  */
    /* 8       8      1          3  |     8          8         1           3                                                  */
    /* 9       9      1          3  |     9          9         1           3                                                  */
    /* 10     10     NA         NA  |    10         10         .           .                                                  */
    /* 11     11      1          4  |    11         11         1           4                                                  */
    /* 12     12     NA         NA  |    12         12         .           .                                                  */
    /* 13     13     NA         NA  |    13         13         .           .                                                  */
    /**************************************************************************************************************************/

    /*____               _   _                             _
    |___ /   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) | | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
            |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete pywant;
    run;quit;

    %utl_pybeginx;
    parmcards4;
    exec(open('c:/oto/fn_pythonx.py').read())
    have,meta = ps.read_sas7bdat('d:/sd1/have.sas7bdat')
    print(have)
    want=pdsql('''
      select
         otr.rownum
        ,otr.censor
        ,(select count(inr.rownum)
      from
         have as inr
      where
         inr.rownum <= otr.rownum and
         inr.censor is null)*censor+1 as na_counter
      from
          have as otr
      order
          by otr.rownum
       ''');
    print(want)
    fn_tosas9x(want,outlib='d:/sd1/',outdsn='pywant',timeest=3);
    ;;;;
    %utl_pyendx;

    proc print data=sd1.pywant;
    run;quit;

    /**************************************************************************************************************************/
    /* PYTHON                          |                                                                                      */
    /*     ROWNUM  CENSOR  na_counter  |  ROWNUM    CENSOR    NA_COUNTE                                                       */
    /*                                 |                                                                                      */
    /* 0      1.0     1.0         1.0  |     1         1           1                                                          */
    /* 1      2.0     NaN         NaN  |     2         .           .                                                          */
    /* 2      3.0     1.0         2.0  |     3         1           2                                                          */
    /* 3      4.0     1.0         2.0  |     4         1           2                                                          */
    /* 4      5.0     NaN         NaN  |     5         .           .                                                          */
    /* 5      6.0     1.0         3.0  |     6         1           3                                                          */
    /* 6      7.0     1.0         3.0  |     7         1           3                                                          */
    /* 7      8.0     1.0         3.0  |     8         1           3                                                          */
    /* 8      9.0     1.0         3.0  |     9         1           3                                                          */
    /* 9     10.0     NaN         NaN  |    10         .           .                                                          */
    /* 10    11.0     1.0         4.0  |    11         1           4                                                          */
    /* 11    12.0     NaN         NaN  |    12         .           .                                                          */
    /* 12    13.0     NaN         NaN  |    13         .           .                                                          */
    /**************************************************************************************************************************/

    /*  _                       _             _
    | || |     _____  _____ ___| |  ___  __ _| |
    | || |_   / _ \ \/ / __/ _ \ | / __|/ _` | |
    |__   _| |  __/>  < (_|  __/ | \__ \ (_| | |
       |_|    \___/_/\_\___\___|_| |___/\__, |_|
                                           |_|
    */

    %utlfkil(d:/xls/wantxl.xlsx);

    %utl_rbeginx;
    parmcards4;
    library(openxlsx)
    library(sqldf)
    library(haven)
    have<-read_sas("d:/sd1/have.sas7bdat")
    wb <- createWorkbook()
    addWorksheet(wb, "have")
    writeData(wb, sheet = "have", x = have)
    saveWorkbook(
        wb
       ,"d:/xls/wantxl.xlsx"
       ,overwrite=TRUE)
    ;;;;
    %utl_rendx;

    %utl_rbeginx;
    parmcards4;
    library(openxlsx)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
     wb<-loadWorkbook("d:/xls/wantxl.xlsx")
     have<-read.xlsx(wb,"have")
     addWorksheet(wb, "want")
     want<-sqldf('
      select
         otr.rownum
        ,otr.censor
        ,(select count(inr.rownum)
      from
         have as inr
      where
         inr.rownum <= otr.rownum and
         inr.censor is null)*censor+1 as na_counter
      from
          have as otr
      order
          by otr.rownum
    ')
     print(want)
     writeData(wb,sheet="want",x=want)
     saveWorkbook(
         wb
        ,"d:/xls/wantxl.xlsx"
        ,overwrite=TRUE)
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /* d:/xls/wantxl.xlsx                                                                                                     */
    /* -----------------------+                                                                                               */
    /* | A1| fx    |ROWNUM    |                                                                                               */
    /* --------------------------------------+                                                                                */
    /* [_] |    A     |    B    |    C       |                                                                                */
    /* --------------------------------------|                                                                                */
    /*  1  |ROWNUM    | CENSOR  |NA_COUNTER  |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  2  |1         | 1       | 1          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  3  |2         | .       | .          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  4  |3         | 1       | 2          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  5  |4         | 1       | 2          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  6  |5         | .       | .          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  7  |6         | 1       | 3          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  8  |7         | 1       | 3          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /*  9  |8         | 1       | 3          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /* 10  |9         | 1       | 3          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /* 11  |10        | .       | .          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /* 12  |11        | 1       | 4          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /* 13  |12        | .       | .          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /* 14  |13        | .       | .          |                                                                                */
    /*  -- |----------+---------+------------|                                                                                */
    /* [WANT]                                                                                                                 */
    /**************************************************************************************************************************/

    /*___                        _       _            _
    | ___|   ___  __ _ ___    __| | __ _| |_ __ _ ___| |_ ___ _ __
    |___ \  / __|/ _` / __|  / _` |/ _` | __/ _` / __| __/ _ \ `_ \
     ___) | \__ \ (_| \__ \ | (_| | (_| | || (_| \__ \ ||  __/ |_) |
    |____/  |___/\__,_|___/  \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                                             |_|
    */

    data want (drop=_:);
      set sd1.have;
      by censor notsorted;
      _na+first.censor and censor^=.;
      if censor^=. then na=_na;
    run;

    /**************************************************************************************************************************/
    /*   CENSOR    NA                                                                                                         */
    /*                                                                                                                        */
    /*      1       1                                                                                                         */
    /*      .       .                                                                                                         */
    /*      1       2                                                                                                         */
    /*      1       2                                                                                                         */
    /*      .       .                                                                                                         */
    /*      1       3                                                                                                         */
    /*      1       3                                                                                                         */
    /*      1       3                                                                                                         */
    /*      1       3                                                                                                         */
    /*      .       .                                                                                                         */
    /*      1       4                                                                                                         */
    /*      .       .                                                                                                         */
    /*      .       .                                                                                                         */
    /**************************************************************************************************************************/

    /*__              _       _           _
     / /_    _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
    | `_ \  | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
    | (_) | | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
     \___/  |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                      |_|
    */

    REPO
    ---------------------------------------------------------------------------------------------------------------------------------------
    https://github.com/rogerjdeangelis/utl-adding-sequence-numbers-and-partitions-in-SAS-sql-without-using-monotonic
    https://github.com/rogerjdeangelis/utl-create-equally-spaced-values-using-partitioning-in-sql-wps-r-python
    https://github.com/rogerjdeangelis/utl-create-primary-key-for-duplicated-records-using-sql-partitionaling-and-pivot-wide-sas-python-r
    https://github.com/rogerjdeangelis/utl-find-first-n-observations-per-category-using-proc-sql-partitioning
    https://github.com/rogerjdeangelis/utl-flag-second-duplicate-using-base-sas-and-sql-sas-python-and-r-partitioning-multi-language
    https://github.com/rogerjdeangelis/utl-incrementing-by-one-for-each-new-group-of-records-sas-r-python-sql-partitioning
    https://github.com/rogerjdeangelis/utl-macro-to-enable-sql-partitioning-by-groups-montonic-first-and-last-dot
    https://github.com/rogerjdeangelis/utl-maintaining-the-orginal-order-while-partitioning-groups-using-sql-partitioning
    https://github.com/rogerjdeangelis/utl-pivot-long-pivot-wide-transpose-partitioning-sql-arrays-wps-r-python
    https://github.com/rogerjdeangelis/utl-pivot-transpose-by-id-using-wps-r-python-sql-using-partitioning
    https://github.com/rogerjdeangelis/utl-top-four-seasonal-precipitation-totals--european-cities-sql-partitions-in-wps-r-python
    https://github.com/rogerjdeangelis/utl-transpose-pivot-wide-using-sql-partitioning-in-wps-r-python
    https://github.com/rogerjdeangelis/utl-transposing-rows-to-columns-using-proc-sql-partitioning
    https://github.com/rogerjdeangelis/utl-transposing-words-into-sentences-using-sql-partitioning-in-r-and-python
    https://github.com/rogerjdeangelis/utl-using-sql-in-wps-r-python-select-the-four-youngest-male-and-female-students-partitioning


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

