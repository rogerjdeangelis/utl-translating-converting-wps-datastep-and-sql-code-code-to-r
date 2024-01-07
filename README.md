# utl-translating-converting-wps-datastep-and-sql-code-code-to-r
Translating converting wps datastep and sql code code to r
    %let pgm=utl-translating-converting-wps-datastep-and-sql-code-code-to-r;

    Translating converting wps datastep and sql code code to r;

    github
    http://tinyurl.com/ywdyh7j4
    https://github.com/rogerjdeangelis/utl-translating-converting-wps-datastep-and-sql-code-code-to-r

    also see

    github
    http://tinyurl.com/526bbdwf
    https://github.com/rogerjdeangelis/utl-converting-common-wps-coding-to-r-and-python

    SOAPBOX ON

     Looping is anathema to R programmers.
     R programmers rather not use this type of looping.
     SQL is also anathema to R and especially to Python programmers.
     Python code was too different for me to include in this translation,.
     It seems that R and especially Python programmers are very porochial.

     Optimization is overrated, expecially for small data(less than 32gb).
     Ram of 128gb/256gb and 64 logical processors or laptops with 32 logical processors and 64gb are not tnat expensive.
    SOAPBOX OFF

     SOLUTIONS

        1 wps wps code
        2 wps R similar code

    also see

    github
    http://tinyurl.com/526bbdwf
    https://github.com/rogerjdeangelis/utl-converting-common-wps-coding-to-r-and-python


    /**************************************************************************************************************************/
    /*                                                |                                                                       */
    /*       __      ___ __  ___                      |      ____                                                             */
    /*       \ \ /\ / / `_ \/ __|                     |     |  _ \                                                            */
    /*        \ V  V /| |_) \__ \                     |     | |_) |                                                           */
    /*         \_/\_/ | .__/|___/                     |     |  _ <                                                            */
    /*                |_|                             |     |_| \_\                                                           */
    /*                                                |                                                                       */
    /*     INPUT:classz.sas7bdat must already exist   |     INPUT:classz.sas7bdat must already exist                          */
    /*                                                |                                                                       */
    /*                                                |                                                                       */
    /*        * Already available                     |     export data=sd1.classz;    # send to R                            */
    /*                                                |     sd1<-"d:/rds/classz.rds";  # like libname                         */
    /*                                                |                                                                       */
    /*        ========================================+=======================================================                */
    /*                                                |                                                                       */
    /*        data classz; * updated;                 |     # datastep;                                                       */
    /*          array hgtwgt height weight;           |     classz$HGTSQR<-Nan;  # retain                                     */
    /*          array hgtwgt height weight;           |     classz$BMI<-NaN;     # retain                                     */
    /*                                                |                                                                       */
    /*          do row=1 to nobs;                     |                                                                       */
    /*                                                |     for(i in 1:nrow(classz))  # _n_  col 5 weight col 4 height        */
    /*            set sd1.classz nobs=nobs point=row; |        {                                                              */
    /*            hgtsqr = hgtwgt[1]*hgtwgt[1];       |         sav<-classz[i,4]^2;                                           */
    /*            bmi    = 703*hgtwgt[2]/hgtsqr;      |         classz[i,6] <- 703*classz[i,5]/sav;                           */
    /*            output;                             |         classz[i,7] <- sav;                                           */
    /*                                                |        };                                                             */                                                                    */
    /*          end;                                  |                                                                       */
    /*                                                |                                                                       */
    /*         =======================================+==========================================================             */
    /*                                                |                                                                       */
    /*                                                |     # proc print;                                                     */
    /*         proc print data=classz;                |     print(classz);                                                    */
    /*                                                |                                                                       */
    /*        ========================================+==========================================================             */
    /*                                                |                                                                       */
    /*        * save permanent updated classz;        |     # save permanent dataset(actually dataframe);                     */
    /*        data sd1.classz; set classz;            |     write_rds(classz,sd1);                                            */
    /*                                                |                                                                       */
    /*        ========================================+==========================================================             */
    /*                                                |                                                                       */
    /*        * reload classz;                        |     # wps data sd1.classz                                             */
    /*        data classz; set sd1.classz;            |     classz<-as.data.frame(read_rds(sd1));                             */
    /*                                                |                                                                       */
    /*        ========================================+==========================================================             */
    /*                                                |                                                                       */
    /*        proc sql;                               |     # wps R proc sql;                                                 */
    /*         create                                 |     maxbmi <-sqldf("                                                  */
    /*           table maxbmi as                      |      select                                                           */
    /*         select                                 |        avg(bmi) as bmiAvg                                             */
    /*           mean(bmi) as bmiAvg                  |       ,max(bmi)  as bmiMax                                            */
    /*          ,max(bmi)  as bmiMax                  |       ,min(bmi)  as bmiMin                                            */
    /*          ,min(bmi)  as bmiMin                  |      from                                                             */
    /*         from                                   |        classz                                                         */
    /*           classz                               |     ");                                                               */
    /*                                                |                                                                       */
    /*                                                |     # wps proc print;                                                 */
    /*         proc print data=maxbmi                 |       print(maxbmi);                                                  */
    /*                                                |                                                                       */
    /*                                                |                                                                       */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    /*----                                                                   ----*/
    /*---- Using classz because class is sometimes a reserved keyword        ----*/
    /*----                                                                   ----*/

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.classz;
      set sashelp.class;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* SD1.CLASSZ total obs=19                                                                                                */
    /*                                                                                                                        */
    /* Obs    NAME       SEX    AGE    HEIGHT    WEIGHT                                                                       */
    /*                                                                                                                        */
    /*   1    Alfred      M      14     69.0      112.5                                                                       */
    /*   2    Alice       F      13     56.5       84.0                                                                       */
    /*   3    Barbara     F      13     65.3       98.0                                                                       */
    /*   4    Carol       F      14     62.8      102.5                                                                       */
    /*   5    Henry       M      14     63.5      102.5                                                                       */
    /*   6    James       M      12     57.3       83.0                                                                       */
    /*   ...                                                                                                                  */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*                            _       _            _                         _
    / | __      ___ __  ___    __| | __ _| |_ __ _ ___| |_ ___ _ __    ___  __ _| |
    | | \ \ /\ / / `_ \/ __|  / _` |/ _` | __/ _` / __| __/ _ \ `_ \  / __|/ _` | |
    | |  \ V  V /| |_) \__ \ | (_| | (_| | || (_| \__ \ ||  __/ |_) | \__ \ (_| | |
    |_|   \_/\_/ | .__/|___/  \__,_|\__,_|\__\__,_|___/\__\___| .__/  |___/\__, |_|
                 |_|                                          |_|             |_|
    */

    %utl_submit_wps64x('

    libname sd1 "d:/sd1";

    options validvarname=any;
    data bmi;
      array hgtwgt height weight;
      do row=1 to nobs;
        set sd1.classz nobs=nobs point=row;
        hgtsqr = hgtwgt[1]*hgtwgt[1];
        bmi    = 703*hgtwgt[2]/hgtsqr;
        output;
      end;
      stop;
    run;quit;

    proc print data=bmi;
    run;quit;

    proc sql;
     create
       table sd1.maxbmi as
     select
       mean(bmi) as bmiAvg
      ,max(bmi)  as bmiMax
      ,min(bmi)  as bmiMin
     from
       bmi
    ;quit;

    proc print data=sd1.maxbmi;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  The WPS System                                                                                                        */
    /*                                bmi                                                                                     */
    /*  Obs    bmiAvg     bmiMax      Min                                                                                     */
    /*                                                                                                                        */
    /*   1     17.8633    21.4297    13.49                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                             _           _ _                                   _
    |___ \  __      ___ __  ___   ___(_)_ __ ___ (_) | __ _ _ __   _ __   ___ ___   __| | ___
      __) | \ \ /\ / / `_ \/ __| / __| | `_ ` _ \| | |/ _` | `__| | `__| / __/ _ \ / _` |/ _ \
     / __/   \ V  V /| |_) \__ \ \__ \ | | | | | | | | (_| | |    | |   | (_| (_) | (_| |  __/
    |_____|   \_/\_/ | .__/|___/ |___/_|_| |_| |_|_|_|\__,_|_|    |_|    \___\___/ \__,_|\___|
                     |_|
    */

    /*----                                                                   ----*/
    /*----  in case you rerun                                                ----*/
    /*----                                                                   ----*/

    %utlfkil(d:/rds/classz.rds);

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    options sasautos=("d:/oto",sasautos);
    proc r;
    export data=sd1.classz r=classz;
    submit;
    library(readr);
    library(sqldf);
    library(tidyverse);

    # wps libname;
    fyl<-"d:/rds/classz.rds";

    # datastep;
    classz$BMI<-NaN;
    classz$HGTSQR<-NaN;

    for(i in 1:nrow(classz))
       {
        sav<-classz[i,4]^2;
        classz[i,6] <- 703*classz[i,5]/sav;
        classz[i,7] <- sav;
       };

    # save permanent dataset(dataframe);

    write_rds(classz,fyl);
    rm(classz);
    ls();

    # wps set command;
    classz<-as.data.frame(read_rds(fyl));

    # wps proc sql;
    maxbmi <-sqldf("
     select
       avg(bmi) as bmiAvg
      ,max(bmi)  as bmiMax
      ,min(bmi)  as bmiMin
     from
       classz
    ");

    # wps proc print;
    maxbmi;

    endsubmit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  The WPS R System                                                                                                      */
    /*                                                                                                                        */
    /*      bmiAvg   bmiMax bmiMin                                                                                            */
    /*  1 17.86325 21.42966  13.49                                                                                            */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
