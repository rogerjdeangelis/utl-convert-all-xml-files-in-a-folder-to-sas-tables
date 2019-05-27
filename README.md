# utl-convert-all-xml-files-in-a-folder-to-sas-tables
Convert all xml files in a folder to sas tables
    Convert all xml files in a folder to sas tables

    github
    https://tinyurl.com/y5vj7xml
    https://github.com/rogerjdeangelis/utl-convert-all-xml-files-in-a-folder-to-sas-tables

    SAS Forum
    https://tinyurl.com/y4e52vqn
    https://communities.sas.com/t5/SAS-Programming/Importing-Multiple-xml-files-from-a-folder/m-p/561693

    No need for proc import


    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

     d:\PARENT
          \
           +---classfit.xml
           |
           +---class.xml
           |
           \---cars.xml


    * make three xml files;

    data _null_;

      if _n_=0 then do;
        %let rc=%sysfunc(dosubl('
            data _null_;
              rc=dcreate("parent","d:/");
              call symputx("folder","parent");
            run;quit;
        '));
       end;

       do dsns='classfit', 'class', 'cars';
         call symputx('dsn',dsns);

         rc=dosubl('
           libname xmlout xml "d:\&folder.\&dsn..xml" xmltype=generic;
              data xmlout.&dsn.;
                 set sashelp.&dsn.;
              run;quit;
           libname xmlout clear;
         ');
       end;

    run;quit;

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    WORK TABLES

      work.class
      work.iris
      work.cars


    proc contents data=work._all_;
    run;quit;


                 Member   Obs, Entries
    #  Name      Type      or Indexes   Vars   File Size  Last Modified

    1  CARS      DATA         428        15        192KB  05/27/2019 10:05:36
    2  CLASS     DATA          19         5        128KB  05/27/2019 10:05:36
    3  CLASSFIT  DATA          19        10        128KB  05/27/2019 10:05:36

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    proc datasets lib=work kill;
    run;quit;

    %symdel memnames folder dsn /nowarn;
    data _null_;

        * get list of xml files in folder;
       if _n_=0 then do;
         %let rc=%sysfunc(dosubl('
             filename dop "d:/parent";
             data contents;
                 length memnames $200 memname $44;
                 retain memnames "";
                 fid=dopen("dop");
                 memcount=dnum(fid);
                 do i=1 to memcount;
                     memname=quote(scan(dread(fid,i),1,"."));
                     memnames=catx(",",memname,memnames);
                 end;
                 call symputx("memnames",memnames);
                 call symputx("folder","parent");
                 rc=dclose(fid);
             run;quit;
             %let mem
         '));
       end;

       length dsns $44; * important because length detemined by fist element;
       do dsns=&memnames;

         call symputx('dsn',dsns);

         rc=dosubl('
           libname xmlinp xml "d:\&folder\&dsn..xml" xmltype=generic;
              proc sql;
                create
                  table &dsn. as
                select
                  *
                from
                  xmlinp.&dsn.
              ;quit;
           libname xmlinp clear;
         ');

        if symget('sqlobs')='0' then do;
           putlog "stopping because " dsn " is empty";
           stop;
        end;

       end;

    run;quit;


