# utl_group_linkage_number_of_times_one_item_has_been_grouped_together_with_another
Group linkage number of times one item has been grouped together with another.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Group linkage number of times one item has been grouped together with another

    Classic linkage problem?
    This is a variation of a matrix DOT product problem.

    Probably best done in R, SAS/IML or WPS/IML?

    THREE SOLUTIONS

           1. Corresp then SQL
           2. Corresp then SQL ARRAY  (least code and very fast?)
           3. Corresp then IML DOT product  (used SAS IML because WPS does not support DOSUBL)
           4. Pure WPS/PROC R  DOT Product
           
           See additional elegant hash solution by Paul on end
           Paul Dorfman <sashole@BELLSOUTH.NET>

    I gave up on a datastep solution with and without 'proc corresp', see end of message for my attempt


    https://stackoverflow.com/questions/51206126/how-can-i-count-the-number-of-times-one-item-has-been-grouped-together-with-anot

    Number of times one item has been grouped together with another

    Dot product has many applications.
    I cant see an easy way to do this in a datastep, you need a matrix language or SQL?


    INPUT
    =====

     WORK HAVE total obs=7

       GROUP     COUNTRY

       Group1      SE
       Group1      DE
       Group2      SE
       Group2      DE
       Group2      FI
       Group3      SE
       Group3      FI

    EXAMPLE OUTPUT
    --------------

      WORK.WANT total obs=3

        VAR1    VAR2    VAL

         DE      FI      1
         DE      SE      2
         FI      SE      2

    RULES
    -----

     CREATE THIS CROSSTAB FIRST

     WORK.HAVTBL total obs=3

        LABEL     DE    FI    SE

        Group1     1     0     1
        Group2     1     1     1
        Group3     0     1     1

     Groups linked for FI and SE

                            |  RULE for FI SE Linkage
         LABEL     FI    SE |  Sum the product
                            |
         Group1     0     1 |  0*1
         Group2     1     1 |  1*1
         Group3     1     1 |  1*1
                            |  ===
                            |    2  Sum FI SE in two groups

        VAR1    VAR2    VAL

         FI      SE      2    ** are in group 2 and 3  (2 groups)


    PROCESS
    =======

    1. CORRESP THEN SQL

       data wantsql (keep=var1 var2 val);

         if _n_=0 then do;
            %let rc=%sysfunc(dosubl('
                ods output observed=havTbl (drop=sum where=(label ne "Sum"));
                proc corresp data=have dim=1 observed;
                   tables group,country;
                run;quit;
                proc sql;
                  proc sql;
                   create
                      table havFat as
                   select
                      sum(DE*FI) as DE_FI
                     ,sum(DE*SE) as DE_SE
                     ,sum(FI*SE) as FI_SE
                   from
                     havTbl
                ;quit;
            '));
         end;

         set havFat;

         array nums _numeric_;
         do over nums;
            var1=scan(vname(nums),1,'_');
            var2=scan(vname(nums),2,'_');
            val=nums;
            output;
         end;

       run;quit;

       /*
       WORK.WANTSQL total obs=3

       VAR1    VAR2    VAL

        DE      FI      1
        DE      SE      2
        FI      SE      2

       Note havFat is also a solution

       Up to 40 obs from HAVFAT total obs=1

       Obs    DE_FI    DE_SE    FI_SE

        1       1        2        2

       */

    2. CORRESP THEN SQL

       ods output observed=havTbl (drop=sum where=(label ne "Sum"));
       proc corresp data=have dim=1 observed;
          tables group,country;
       run;quit;
       %array(mul,values=DE*FI DE*SE FI*SE)
       %array(namOne,values=DE DE FI SE)
       %array(namTwo,values=FI SE SE)
         proc sql;
           create
             table havDov as
           %do_over(mul namOne namTwo, phrase=%str(
           select
              "?namOne" as var1
             ,"?namTwo" as var2
             ,sum(?mul) as val from havTbl)
             ,between=union
           )
       ;quit;


    3. CORRESP THEN IML DOT PRODUCT

       &* my IML is very rusty;
       data wantiml (keep=var1 var2 val);

         if _n_=0 then do;
            %let rc=%sysfunc(dosubl('
                ods output observed=havTbl (drop=sum where=(label ne "Sum"));
                proc corresp data=have dim=1 observed;
                   tables group,country;
                run;quit;
                proc iml;
                  use havTbl;
                  read all var _NUM_ into X[c=varNames];
                  dot=X`*X;
                  create havDot from dot[c=varNames];
                  append from dot;
                run;quit;
            '));
         end;

         set havDot;

         array nums _numeric_;

         select (_n_);
           when (1) do;
              var1=vname(nums[1]);var2=vname(nums[2]);val=nums[2];output;
              var1=vname(nums[1]);var2=vname(nums[3]);val=nums[3];output;
           end;
           when (2) do;
              var1=vname(nums[2]);var2=vname(nums[3]);val=nums[3];output;
           end;
           otherwise;
         end;

       run;quit;

       The IML Dot product is more useful than the reduced solution
       HAVDOT total obs=3

          DE    FI    SE

       DE  2     1     2
       FI  1     2     2
       SE  2     2     3

       The ops solution is a subset of the dot product

      ANTIML total obs=3

        VAR1    VAR2    VAL

         DE      FI      1
         DE      SE      2
         FI      SE      2

    4. PURE WPS PROC R DOT PRODUCT (WORKING CODE)


        m1 <- crossprod(table(have));
        m1[lower.tri(m1, diag = TRUE)] <- NA;
        wantwps<-subset(as.data.frame.table(m1), !is.na(Freq));

    OUTPUT
    ======


     SQL WORK.WANT total obs=3

        VAR1    VAR2    VAL

         DE      FI      1
         DE      SE      2
         FI      SE      2


     R WORK.WANTWPS total obs=3

                 COUNTRY_
      COUNTRY       1        FREQ

        DE          FI         1
        DE          SE         2
        FI          SE         2

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    data have;
       input group$ country$;
    cards4;
    Group1      SE
    Group1      DE
    Group2      SE
    Group2      DE
    Group2      FI
    Group3      SE
    Group3      FI
    ;;;;
    run;quit;


    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    For SAS see process (SPS does not support DOSUBL)

    * WPS Proc R;

    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk "%sysfunc(pathname(work))";
    libname hlp "C:\Program Files\SASHome\SASFoundation\9.4\core\sashelp";
    data sd1.have;
      set wrk.have;
    run;quit;
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(haven);
    have<-read_sas("d:/sd1/have.sas7bdat");
    head(have);
    m1 <- crossprod(table(have));
    m1[lower.tri(m1, diag = TRUE)] <- NA;
    wantwps<-subset(as.data.frame.table(m1), !is.na(Freq));
    wantwps;
    endsubmit;
    import r=wantwps data=wrk.wantwps;
    run;quit;
    ');

    Up to 40 obs from

     WANTWPS total obs=3

                 COUNTRY_
      COUNTRY       1        FREQ

        DE          FI         1
        DE          SE         2
        FI          SE         2

    *    _       _            _                __       _ _
      __| | __ _| |_ __ _ ___| |_ ___ _ __    / _| __ _(_) |
     / _` |/ _` | __/ _` / __| __/ _ \ '_ \  | |_ / _` | | |
    | (_| | (_| | || (_| \__ \ ||  __/ |_) | |  _| (_| | | |
     \__,_|\__,_|\__\__,_|___/\__\___| .__/  |_|  \__,_|_|_|
                                     |_|
    ;
    ods output observed=havTbl(drop=sum where=(label ne 'Sum'));
    proc corresp data=have dim=1 observed;
    tables group,country;
    run;quit;

    data wantDat;
       if _n_=0 then do;
          %let rc=%sysfunc(dosubl('
              ods output observed=havTbl (drop=sum where=(label ne "Sum"));
              proc corresp data=have dim=1 observed;
                 tables group,country;
              run;quit;
              proc sql;select count(*) into :obs trimmed from havTbl;quit;
          '));
       end;
       set havTbl end=dne;
       array nums[*] _numeric_;
       dimnum=dim(nums);
       retain val1-val&obs;
       array vals[&obs] val1-val&obs ;
       array vars[&obs] $32 var1=var&obs;
       retain idx 0;
       do lft=1 to dim(nums)-1;
         do ryt=2 to dim(nums);
           if lft ne ryt then do;
             idx=idx+1;
             vals[idx]=sum(vals[idx],nums[lft]*nums[ryt]);
             var[lft] =varname(nums[lft]);var[ryt] =varname(nums[ryt]);
             putlog (_all_) ($ = /) /;
           end;
         end;
       end;
       idx=0;
       if dne then do;
          do ix=1 to 3;
            var[

    run;quit;




    See Elegant hash solution by Paul on end
    Paul Dorfman <sashole@BELLSOUTH.NET>

    *____             _
    |  _ \ __ _ _   _| |
    | |_) / _` | | | | |
    |  __/ (_| | |_| | |
    |_|   \__,_|\__,_|_|

    ;
    It shouldn't be difficult to do it using the hash object. The only subtleties are:

    (a) two iterators are needed for table H to pair the values of COUNTRY
    before storing the value pairs in table V together with the requisite counts.
    (b) some index gymnastics to ensure that the pairing is going one-way.

    The duplicate values of COUNTRY within each BY group are killed automatically
    by table H. If such dupes should need to be accounted for,
    the argument tag MULTIDATA:"Y" could be added.

    data have ;
      input group $ country $ ;
      cards ;
    Group1  SE
    Group1  DE
    Group2  SE
    Group2  DE
    Group2  FI
    Group3  SE
    Group3  FI
    ;
    run ;

    data _null_ ;
      if _n_ = 1 then do ;
        dcl hash h (ordered:"a") ;
        h.definekey ("country") ;
        h.definedone () ;
        dcl hiter i ("h") ;
        dcl hiter j ("h") ;
        dcl hash v (ordered:"a") ;
        v.definekey  ("var1", "var2") ;
        v.definedata ("var1", "var2", "val") ;
        v.definedone () ;
      end ;
      do until (last.group) ;
        set have end = eof ;
        by group ;
        h.ref() ;
      end ;
      do _i = 1 by 1 while (i.next() = 0) ;
        var1 = country ;
        do _j = 1 by 1 while (j.next() = 0) ;
          if _j <= _i then continue ;
          var2 = country ;
          if v.find() ne 0 then val = 1 ;
          else                  val + 1 ;
          v.replace() ;
        end ;
      end ;
      h.clear() ;
      if eof then v.output (dataset:"want") ;
    run ;

    Another way of doing the same would be to leave the table H only with the
    function of unduplicating COUNTRY within each GROUP and use an array to do
    the pairing. If the pairs were numerous, I suspect it would be quite a bit
    efficient than using two iterators. The trade-off is the need to pre-comute
    the size of the array at the expense of an extra pass through the input or allocate
    it as "big enough" (as it is done below).

    data _null_ ;
      if _n_ = 1 then do ;
        dcl hash h (ordered:"a") ;
        h.definekey ("country") ;
        h.definedone () ;
        dcl hiter ih ("h") ;
        dcl hash v (ordered:"a") ;
        v.definekey  ("var1", "var2") ;
        v.definedata ("var1", "var2", "val") ;
        v.definedone () ;
      end ;
      array vv [100] $ 2 _temporary_ ;
      do until (last.group) ;
        set have end = eof ;
        by group ;
        if h.check() = 0 then continue ;
        n = sum (n, 1) ;
        vv[n] = country ;
        h.add() ;
      end ;
      h.clear() ;
      do i = 1 to n - 1 ;
        var1 = vv[i] ;
        do j = i + 1 to n ;
          var2 = vv[j] ;
          if v.find() ne 0 then val = 1 ;
          else                  val + 1 ;
          v.replace() ;
        end ;
      end ;
      if eof then v.output (dataset:"want") ;
    run ;

    Finally, in the steps above, advantage is taken of the fact that the input is sorted by GROUP.
    However, it's not difficult to recode them in order to get rid of this prerequisite.

    Best regards,

    Paul Dorfman

