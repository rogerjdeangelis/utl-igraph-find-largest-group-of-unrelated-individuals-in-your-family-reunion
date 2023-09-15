# utl-igraph-find-largest-group-of-unrelated-individuals-in-your-family-reunion
Find largest group of unrelated individuals in your family reunion    
    %let pgm=utl-igraph-find-largest-group-of-unrelated-individuals-in-your-family-reunion;

    Find largest group of unrelated individuals in your family reunion

    github
    https://tinyurl.com/cbww463w
    https://github.com/rogerjdeangelis/utl-igraph-find-largest-group-of-unrelated-individuals-in-your-family-reunion

    stackoverflow R
    https://tinyurl.com/aswkfa3t
    https://stackoverflow.com/questions/77107074/from-a-pairwise-matrix-find-the-largest-group-of-individuals-that-equal-a-certa

    Soltion by
    https://stackoverflow.com/users/9463489/jblood94

    'I have a pairwise relatedness 26x26 matrix containing the relatedness values for all
    pairwise combinations of 26 individuals. I would like to find the largest group of
    individuals that are completely unrelated, i.e. where all pairwise
    relatedness values i the group equal 0.'

    The most difficult part of this solution is converting the very comples
    R only data structures to a somewhat usable dataframe.

    /***********************************************************************************************************************************
    /*                                                       |          |
    /* INPUT                                                 | RULE     | OUTPUT EXAMPLE
    /*                                                       |          |
    /*   A B C D E F G H I J K L M N O P Q R S T U V W X Y Z |          |    I  P  W  X    Largest Unrelated grouo of individuals
    /*                                                       |          |
    /* a . 0 1 2 1 2 0 2 2 0 1 2 0 1 2 2 0 0 0 0 0 2 2 2 1 1 | Find     | I  NA  0  0  0   I is unrelated to P, W, and X  (all 0)
    /* b 0 . 0 1 2 2 0 2 0 2 0 2 2 2 1 2 2 1 1 0 1 2 1 2 0 1 | largest  | P   0 NA  0  0   P is unrelated to I, W, and X
    /* c 1 0 . 0 1 0 2 1 2 1 0 1 0 1 2 2 2 2 1 2 2 0 2 0 1 0 | group of | W   0  0 NA  0   X is unrelated to I, W, and P
    /* d 2 1 0 . 2 2 2 2 2 2 1 1 0 1 2 1 2 2 1 2 1 0 1 0 2 1 | unrelated| X   0  0  0 NA   W is unrelated to I, X, and P
    /* e 1 2 1 2 . 2 1 0 1 0 1 0 0 0 1 2 0 2 0 2 2 1 2 1 2 2 | relatives|
    /* f 2 2 0 2 2 . 2 2 2 1 1 2 1 2 0 2 0 2 2 0 1 1 0 2 2 2 |          |                   I               P               W  X
    /* g 0 0 2 2 1 2 . 0 2 1 2 2 2 2 0 1 2 0 2 1 0 0 1 1 2 1 |          |                   *               *               *  *
    /* h 2 2 1 2 0 2 0 . 2 2 1 0 2 2 1 0 1 1 1 1 2 1 1 1 1 2 |          |                   ==              ==             ==  ==
    /* i 2 0 2 2 1 2 2 2 . 1 2 1 0 2 2 0 2 2 2 0 2 0 0 0 0 2 |          |   a b c d e f g h  i  j k l m n o  p  q r s t u v  w  x  y z
    /* j 0 2 1 2 0 1 1 2 1 . 1 1 2 2 0 0 1 1 2 2 2 1 0 0 2 2 |          |
    /* k 1 0 0 1 1 1 2 1 2 1 . 2 2 1 0 0 2 0 2 0 0 1 1 1 1 2 |          | I 2 0 2 2 1 2 2 2  .  1 2 1 0 2 2  0  2 2 2 0 2 0  0  0  0 2
    /* l 2 2 1 1 0 2 2 0 1 1 2 . 1 1 2 0 2 2 1 2 1 0 0 2 1 1 |          | P 2 2 2 1 2 2 1 0  0  0 0 0 2 0 2  .  2 2 2 1 0 2  0  0  1 2
    /* m 0 2 0 0 0 1 2 2 0 2 2 1 . 0 2 2 0 2 1 1 1 1 0 2 1 1 |          | W 2 1 2 1 2 0 1 1  0  0 1 0 0 0 2  0  1 2 2 1 2 2  .  0  2 0
    /* n 1 2 1 1 0 2 2 2 2 2 1 1 0 . 1 0 1 2 1 2 0 1 0 1 1 2 |          | X 2 2 0 0 1 2 1 1  0  0 1 2 2 1 1  0  0 2 1 2 0 0  0  .  1 2
    /* o 2 1 2 2 1 0 0 1 2 0 0 2 2 1 . 2 2 0 1 2 1 2 2 1 1 0 |          |
    /* p 2 2 2 1 2 2 1 0 0 0 0 0 2 0 2 . 2 2 2 1 0 2 0 0 1 2 |          |
    /* q 0 2 2 2 0 0 2 1 2 1 2 2 0 1 2 2 . 1 0 1 2 2 1 0 1 1 |          |
    /* r 0 1 2 2 2 2 0 1 2 1 0 2 2 2 0 2 1 . 1 1 2 1 2 2 2 1 |          |
    /* s 0 1 1 1 0 2 2 1 2 2 2 1 1 1 1 2 0 1 . 2 1 1 2 1 1 1 |          |
    /* t 0 0 2 2 2 0 1 1 0 2 0 2 1 2 2 1 1 1 2 . 0 0 1 2 2 0 |          |
    /* u 0 1 2 1 2 1 0 2 2 2 0 1 1 0 1 0 2 2 1 0 . 2 2 0 2 0 |          |
    /* v 2 2 0 0 1 1 0 1 0 1 1 0 1 1 2 2 2 1 1 0 2 . 2 0 1 1 |          |
    /* w 2 1 2 1 2 0 1 1 0 0 1 0 0 0 2 0 1 2 2 1 2 2 . 0 2 0 |          |
    /* x 2 2 0 0 1 2 1 1 0 0 1 2 2 1 1 0 0 2 1 2 0 0 0 . 1 2 |          |
    /* y 1 0 1 2 2 2 2 1 0 2 1 1 1 1 1 1 1 2 1 2 2 1 2 1 . 0 |          |
    /* z 1 1 0 1 2 2 1 2 2 2 2 1 1 2 0 2 1 1 1 0 0 1 0 2 0 . |          |
    /*                                                       |          |
    /***********************************************************************************************************************************

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input a b c d e f g h i j k l m n o p q r s t u v w x y z;
    cards4;
    . 0 1 2 1 2 0 2 2 0 1 2 0 1 2 2 0 0 0 0 0 2 2 2 1 1
    0 . 0 1 2 2 0 2 0 2 0 2 2 2 1 2 2 1 1 0 1 2 1 2 0 1
    1 0 . 0 1 0 2 1 2 1 0 1 0 1 2 2 2 2 1 2 2 0 2 0 1 0
    2 1 0 . 2 2 2 2 2 2 1 1 0 1 2 1 2 2 1 2 1 0 1 0 2 1
    1 2 1 2 . 2 1 0 1 0 1 0 0 0 1 2 0 2 0 2 2 1 2 1 2 2
    2 2 0 2 2 . 2 2 2 1 1 2 1 2 0 2 0 2 2 0 1 1 0 2 2 2
    0 0 2 2 1 2 . 0 2 1 2 2 2 2 0 1 2 0 2 1 0 0 1 1 2 1
    2 2 1 2 0 2 0 . 2 2 1 0 2 2 1 0 1 1 1 1 2 1 1 1 1 2
    2 0 2 2 1 2 2 2 . 1 2 1 0 2 2 0 2 2 2 0 2 0 0 0 0 2
    0 2 1 2 0 1 1 2 1 . 1 1 2 2 0 0 1 1 2 2 2 1 0 0 2 2
    1 0 0 1 1 1 2 1 2 1 . 2 2 1 0 0 2 0 2 0 0 1 1 1 1 2
    2 2 1 1 0 2 2 0 1 1 2 . 1 1 2 0 2 2 1 2 1 0 0 2 1 1
    0 2 0 0 0 1 2 2 0 2 2 1 . 0 2 2 0 2 1 1 1 1 0 2 1 1
    1 2 1 1 0 2 2 2 2 2 1 1 0 . 1 0 1 2 1 2 0 1 0 1 1 2
    2 1 2 2 1 0 0 1 2 0 0 2 2 1 . 2 2 0 1 2 1 2 2 1 1 0
    2 2 2 1 2 2 1 0 0 0 0 0 2 0 2 . 2 2 2 1 0 2 0 0 1 2
    0 2 2 2 0 0 2 1 2 1 2 2 0 1 2 2 . 1 0 1 2 2 1 0 1 1
    0 1 2 2 2 2 0 1 2 1 0 2 2 2 0 2 1 . 1 1 2 1 2 2 2 1
    0 1 1 1 0 2 2 1 2 2 2 1 1 1 1 2 0 1 . 2 1 1 2 1 1 1
    0 0 2 2 2 0 1 1 0 2 0 2 1 2 2 1 1 1 2 . 0 0 1 2 2 0
    0 1 2 1 2 1 0 2 2 2 0 1 1 0 1 0 2 2 1 0 . 2 2 0 2 0
    2 2 0 0 1 1 0 1 0 1 1 0 1 1 2 2 2 1 1 0 2 . 2 0 1 1
    2 1 2 1 2 0 1 1 0 0 1 0 0 0 2 0 1 2 2 1 2 2 . 0 2 0
    2 2 0 0 1 2 1 1 0 0 1 2 2 1 1 0 0 2 1 2 0 0 0 . 1 2
    1 0 1 2 2 2 2 1 0 2 1 1 1 1 1 1 1 2 1 2 2 1 2 1 . 0
    1 1 0 1 2 2 1 2 2 2 2 1 1 2 0 2 1 1 1 0 0 1 0 2 0 .
    ;;;;
    run;quit;

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    /*---- for development interactively                                     ----*/
    proc datasets lib=sd1 nolist nodetails;
    delete wantpre want1 want2 want3; run;quit;
    %symdel rowcol groups individual y1 y2 y3 / nowarn;

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    library(igraph);
    m<-as.matrix(have);
    m <- matrix(m, rep(list(letters), 2));
    m[lower.tri(m)] <- t(m)[lower.tri(m)];
    diag(m) <- NA;
    is <- largest_ivs(graph_from_adjacency_matrix(m, "undirected"));
    is;
    want<-as.data.frame(lapply(is, \(i) m[i, i]));
    want;
    writeClipboard(paste(as.character(ncol(want)/nrow(want)),as.character(nrow(want))));
    endsubmit;
    import data=sd1.wantpre r=want;
    run;quit;
    proc print data=sd1.wantpre;
    run;quit;
    ',return=rowcol);

    %let groups      = %scan(&rowcol,1);
    %let individuals = %scan(&rowcol,2);

    /*---- sentinels to slice the out the 3 4x4 largest unrelated relatives ----*/
    data _null_;
      array vrs[&groups ,&individuals] $8 (%utl_varlist(data=sd1.wantpre,Qstyle=DOUBLE,Od=%str(,)  ) );
      do i=1 to &groups;
           call symputx(cats('y',i) , catx("--",vrs[i,1],vrs[i,&individuals]));
      end;
    run;quit;

    data sd1.want1;set sd1.wantpre(keep=&y1); run;quit;
    data sd1.want2;set sd1.wantpre(keep=&y2); run;quit;
    data sd1.want3;set sd1.wantpre(keep=&y3); run;quit;


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* R output                                                                                                               */
    /*                                                                                                                        */
    /* [[1]]                                                                                                                  */
    /* [1] I P W X                                                                                                            */
    /* [[2]]                                                                                                                  */
    /* [1] C D V X                                                                                                            */
    /* [[3]]                                                                                                                  */
    /* [1] J P W X                                                                                                            */
    /*                                                                                                                        */
    /*    I  P  W  X  C  D  V X.1  J P.1 W.1 X.2                                                                              */
    /* 1 NA  0  0  0 NA  0  0   0 NA   0   0   0                                                                              */
    /* 2  0 NA  0  0  0 NA  0   0  0  NA   0   0                                                                              */
    /* 3  0  0 NA  0  0  0 NA   0  0   0  NA   0                                                                              */
    /* 4  0  0  0 NA  0  0  0  NA  0   0   0  NA                                                                              */
    /*                                                                                                                        */
    /* WPS                                                                                                                    */
    /*                                                                                                                        */
    /*  SD1.WANT1                                                                                                             */
    /*                                                                                                                        */
    /*  Obs    I    P    W    X                                                                                               */
    /*                                                                                                                        */
    /*   1     .    0    0    0                                                                                               */
    /*   2     0    .    0    0                                                                                               */
    /*   3     0    0    .    0                                                                                               */
    /*   4     0    0    0    .                                                                                               */
    /*                                                                                                                        */
    /*  SD1.WANT2                                                                                                             */
    /*                                                                                                                        */
    /* Obs    C    D    V     X                                                                                              */
    /*                                                                                                                        */
    /*  1     .    0    0     0                                                                                               */
    /*  2     0    .    0     0                                                                                               */
    /*  3     0    0    .     0                                                                                               */
    /*  4     0    0    0     .                                                                                               */
    /*                                                                                                                        */
    /*  SD1.WANT3                                                                                                             */
    /*                                                                                                                        */
    /* Obs    J     P      W      X                                                                                           */
    /*                                                                                                                        */
    /*  1     .     0      0      0                                                                                           */
    /*  2     0     .      0      0                                                                                           */
    /*  3     0     0      .      0                                                                                           */
    /*  4     0     0      0      .                                                                                           */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
