    %let pgm=utl-igraph-find-largest-group-of-unrelated-individuals-in-your-family-reunion;

    Find largest group of unrelated individuals in your family reunion

     Two Solutions
         1 R

         2 SAS/OR Proc Optmodel
           Looks like WPS has not added OR to base (yet?)
           SAS proc optmodel
           https://mail.google.com/mail/u/0/?tab=rm&ogbl#inbox/FMfcgzGtxKLnSvQxtKNhzqFWQvmGFbzk
           Rob Pratt
           00000c5046759d08-dmarc-request@listserv.uga.edu
           SAS-L: https://listserv.uga.edu/scripts/wa-UGA.exe?A2=SAS-L;64258a98.2309C&S=

         3 Python has IGRAPH (solution not posted)

    'I have a pairwise relatedness 26x26 matrix containing the relatedness values for all
    pairwise combinations of 26 individuals. I would like to find the largest group of
    individuals that are completely unrelated, i.e. where all pairwise
    relatedness values i the group equal 0.'

    github
    https://tinyurl.com/cbww463w
    https://github.com/rogerjdeangelis/utl-igraph-find-largest-group-of-unrelated-individuals-in-your-family-reunion

    stackoverflow R
    https://tinyurl.com/aswkfa3t
    https://stackoverflow.com/questions/77107074/from-a-pairwise-matrix-find-the-largest-group-of-individuals-that-equal-a-certa

    SAS-L

    Solutions by
    jbood94
    https://stackoverflow.com/users/9463489/jblood94

    Rob Pratt
    00000c5046759d08-dmarc-request@listserv.uga.edu

    Comments bt Rob and Mark

    Mark Keintz

    Would converting all the 1's and 2's to zero and converting all the 0's to 1's, and
    then treating it as a distance matrix to be analyzed by OPTNET work, by asking optnet
    for the largest connected collection of nodes?

    And, if so, would that scale up any differently than the OPTMODEL solution?

    Rob Pratt

    The problem is to find a maximum independent set.  As you noted, that is equivalent to finding a maximum clique in the complement.
    PROC OPTNET will find (all or a fixed number of) maximal cliques,
    and you can access the same algorithm via the network solver in PROC OPTMODEL:

    So we see that there are 49 maximal cliques, and 3 of these are maximum cliques.
    But finding all maximal cliques to find a maximum clique is generally inefficient.

    Roger

    The most difficult part of the R solution is converting the very complex
    R only data structures to a somewhat usable dataframe.

    /************************************************************************************************************************************/
    /*                                                       |          |                                                               */
    /* INPUT                                                 | RULE     | OUTPUT EXAMPLE                                                */
    /*                                                       |          |                                                               */
    /*   A B C D E F G H I J K L M N O P Q R S T U V W X Y Z |          |    I  P  W  X    Largest Unrelated grouo of individuals       */
    /*                                                       |          |                                                               */
    /* a . 0 1 2 1 2 0 2 2 0 1 2 0 1 2 2 0 0 0 0 0 2 2 2 1 1 | Find     | I  NA  0  0  0   I is unrelated to P, W, and X  (all 0)       */
    /* b 0 . 0 1 2 2 0 2 0 2 0 2 2 2 1 2 2 1 1 0 1 2 1 2 0 1 | largest  | P   0 NA  0  0   P is unrelated to I, W, and X                */
    /* c 1 0 . 0 1 0 2 1 2 1 0 1 0 1 2 2 2 2 1 2 2 0 2 0 1 0 | group of | W   0  0 NA  0   X is unrelated to I, W, and P                */
    /* d 2 1 0 . 2 2 2 2 2 2 1 1 0 1 2 1 2 2 1 2 1 0 1 0 2 1 | unrelated| X   0  0  0 NA   W is unrelated to I, X, and P                */
    /* e 1 2 1 2 . 2 1 0 1 0 1 0 0 0 1 2 0 2 0 2 2 1 2 1 2 2 | relatives|                                                               */
    /* f 2 2 0 2 2 . 2 2 2 1 1 2 1 2 0 2 0 2 2 0 1 1 0 2 2 2 |          |                   I               P               W  X        */
    /* g 0 0 2 2 1 2 . 0 2 1 2 2 2 2 0 1 2 0 2 1 0 0 1 1 2 1 |          |                   *               *               *  *        */
    /* h 2 2 1 2 0 2 0 . 2 2 1 0 2 2 1 0 1 1 1 1 2 1 1 1 1 2 |          |                   ==              ==             ==  ==       */
    /* i 2 0 2 2 1 2 2 2 . 1 2 1 0 2 2 0 2 2 2 0 2 0 0 0 0 2 |          |   a b c d e f g h  i  j k l m n o  p  q r s t u v  w  x  y z  */
    /* j 0 2 1 2 0 1 1 2 1 . 1 1 2 2 0 0 1 1 2 2 2 1 0 0 2 2 |          |                                                               */
    /* k 1 0 0 1 1 1 2 1 2 1 . 2 2 1 0 0 2 0 2 0 0 1 1 1 1 2 |          | I 2 0 2 2 1 2 2 2  .  1 2 1 0 2 2  0  2 2 2 0 2 0  0  0  0 2  */
    /* l 2 2 1 1 0 2 2 0 1 1 2 . 1 1 2 0 2 2 1 2 1 0 0 2 1 1 |          | P 2 2 2 1 2 2 1 0  0  0 0 0 2 0 2  .  2 2 2 1 0 2  0  0  1 2  */
    /* m 0 2 0 0 0 1 2 2 0 2 2 1 . 0 2 2 0 2 1 1 1 1 0 2 1 1 |          | W 2 1 2 1 2 0 1 1  0  0 1 0 0 0 2  0  1 2 2 1 2 2  .  0  2 0  */
    /* n 1 2 1 1 0 2 2 2 2 2 1 1 0 . 1 0 1 2 1 2 0 1 0 1 1 2 |          | X 2 2 0 0 1 2 1 1  0  0 1 2 2 1 1  0  0 2 1 2 0 0  0  .  1 2  */
    /* o 2 1 2 2 1 0 0 1 2 0 0 2 2 1 . 2 2 0 1 2 1 2 2 1 1 0 |          |                                                               */
    /* p 2 2 2 1 2 2 1 0 0 0 0 0 2 0 2 . 2 2 2 1 0 2 0 0 1 2 |          |                                                               */
    /* q 0 2 2 2 0 0 2 1 2 1 2 2 0 1 2 2 . 1 0 1 2 2 1 0 1 1 |          |                                                               */
    /* r 0 1 2 2 2 2 0 1 2 1 0 2 2 2 0 2 1 . 1 1 2 1 2 2 2 1 |          |                                                               */
    /* s 0 1 1 1 0 2 2 1 2 2 2 1 1 1 1 2 0 1 . 2 1 1 2 1 1 1 |          |                                                               */
    /* t 0 0 2 2 2 0 1 1 0 2 0 2 1 2 2 1 1 1 2 . 0 0 1 2 2 0 |          |                                                               */
    /* u 0 1 2 1 2 1 0 2 2 2 0 1 1 0 1 0 2 2 1 0 . 2 2 0 2 0 |          |                                                               */
    /* v 2 2 0 0 1 1 0 1 0 1 1 0 1 1 2 2 2 1 1 0 2 . 2 0 1 1 |          |                                                               */
    /* w 2 1 2 1 2 0 1 1 0 0 1 0 0 0 2 0 1 2 2 1 2 2 . 0 2 0 |          |                                                               */
    /* x 2 2 0 0 1 2 1 1 0 0 1 2 2 1 1 0 0 2 1 2 0 0 0 . 1 2 |          |                                                               */
    /* y 1 0 1 2 2 2 2 1 0 2 1 1 1 1 1 1 1 2 1 2 2 1 2 1 . 0 |          |                                                               */
    /* z 1 1 0 1 2 2 1 2 2 2 2 1 1 2 0 2 1 1 1 0 0 1 0 2 0 . |          |                                                               */
    /*                                                       |          |                                                               */
    /************************************************************************************************************************************/

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

    /*   ____
    / | |  _ \
    | | | |_) |
    | | |  _ <
    |_| |_| \_\

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

    Select igraph from d:/git/git_010_repos.sasbdat

    REPO
    -----------------------------------------------------------------------------------------------------------------------
    https://github.com/rogerjdeangelis/utl-R-AI-igraph-list-connections-in-a-non-directed-graph-for-a-subset-of-vertices
    https://github.com/rogerjdeangelis/utl-how-many-triangles-in-the-polygon-r-igraph-AI
    https://github.com/rogerjdeangelis/utl-identify-linked-and-unliked-paths-r-igraph
    https://github.com/rogerjdeangelis/utl-shortest-and-longest-travel-time-from-home-to-work-igraph-AI
    https://github.com/rogerjdeangelis/utl_remove_isolated_nodes_from_an_network_r_igraph

    /*___                     __
    |___ \   ___  __ _ ___   / /__  _ __
      __) | / __|/ _` / __| / / _ \| `__|
     / __/  \__ \ (_| \__ \/ / (_) | |
    |_____| |___/\__,_|___/_/ \___/|_|

    */

    options validvarname=v7;
    data have;
     input node$ a b c d e f g h i j k l m n o p q r s t u v w x y z;
    cards4;
    a . 0 1 2 1 2 0 2 2 0 1 2 0 1 2 2 0 0 0 0 0 2 2 2 1 1
    b 0 . 0 1 2 2 0 2 0 2 0 2 2 2 1 2 2 1 1 0 1 2 1 2 0 1
    c 1 0 . 0 1 0 2 1 2 1 0 1 0 1 2 2 2 2 1 2 2 0 2 0 1 0
    d 2 1 0 . 2 2 2 2 2 2 1 1 0 1 2 1 2 2 1 2 1 0 1 0 2 1
    e 1 2 1 2 . 2 1 0 1 0 1 0 0 0 1 2 0 2 0 2 2 1 2 1 2 2
    f 2 2 0 2 2 . 2 2 2 1 1 2 1 2 0 2 0 2 2 0 1 1 0 2 2 2
    g 0 0 2 2 1 2 . 0 2 1 2 2 2 2 0 1 2 0 2 1 0 0 1 1 2 1
    h 2 2 1 2 0 2 0 . 2 2 1 0 2 2 1 0 1 1 1 1 2 1 1 1 1 2
    i 2 0 2 2 1 2 2 2 . 1 2 1 0 2 2 0 2 2 2 0 2 0 0 0 0 2
    j 0 2 1 2 0 1 1 2 1 . 1 1 2 2 0 0 1 1 2 2 2 1 0 0 2 2
    k 1 0 0 1 1 1 2 1 2 1 . 2 2 1 0 0 2 0 2 0 0 1 1 1 1 2
    l 2 2 1 1 0 2 2 0 1 1 2 . 1 1 2 0 2 2 1 2 1 0 0 2 1 1
    m 0 2 0 0 0 1 2 2 0 2 2 1 . 0 2 2 0 2 1 1 1 1 0 2 1 1
    n 1 2 1 1 0 2 2 2 2 2 1 1 0 . 1 0 1 2 1 2 0 1 0 1 1 2
    o 2 1 2 2 1 0 0 1 2 0 0 2 2 1 . 2 2 0 1 2 1 2 2 1 1 0
    p 2 2 2 1 2 2 1 0 0 0 0 0 2 0 2 . 2 2 2 1 0 2 0 0 1 2
    q 0 2 2 2 0 0 2 1 2 1 2 2 0 1 2 2 . 1 0 1 2 2 1 0 1 1
    r 0 1 2 2 2 2 0 1 2 1 0 2 2 2 0 2 1 . 1 1 2 1 2 2 2 1
    s 0 1 1 1 0 2 2 1 2 2 2 1 1 1 1 2 0 1 . 2 1 1 2 1 1 1
    t 0 0 2 2 2 0 1 1 0 2 0 2 1 2 2 1 1 1 2 . 0 0 1 2 2 0
    u 0 1 2 1 2 1 0 2 2 2 0 1 1 0 1 0 2 2 1 0 . 2 2 0 2 0
    v 2 2 0 0 1 1 0 1 0 1 1 0 1 1 2 2 2 1 1 0 2 . 2 0 1 1
    w 2 1 2 1 2 0 1 1 0 0 1 0 0 0 2 0 1 2 2 1 2 2 . 0 2 0
    x 2 2 0 0 1 2 1 1 0 0 1 2 2 1 1 0 0 2 1 2 0 0 0 . 1 2
    y 1 0 1 2 2 2 2 1 0 2 1 1 1 1 1 1 1 2 1 2 2 1 2 1 . 0
    z 1 1 0 1 2 2 1 2 2 2 2 1 1 2 0 2 1 1 1 0 0 1 0 2 0 .
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                           _                                                                            */
    /*   ___ ___  _ __ ___  _ __ ___   ___ _ __ | |_ ___                                                                      */
    /*  / __/ _ \| `_ ` _ \| `_ ` _ \ / _ \ `_ \| __/ __|                                                                     */
    /* | (_| (_) | | | | | | | | | | |  __/ | | | |_\__ \                                                                     */
    /*  \___\___/|_| |_| |_|_| |_| |_|\___|_| |_|\__|___/                                                                     */
    /*                                                                                                                        */
    /*  Mark Keintz                                                                                                           */
    /*  mkeintz@outlook.com                                                                                                   */
    /*                                                                                                                        */
    /*  Would converting all the 1's and 2's to zero and converting all the 0's to 1's, and                                   */
    /*  then treating it as a distance matrix to be analyzed by OPTNET work, by asking optnet                                 */
    /*  for the largest connected collection of nodes?                                                                        */
    /*                                                                                                                        */
    /*  And, if so, would that scale up any differently than the OPTMODEL solution?                                           */
    /*                                                                                                                        */
    /*  Rob Pratt                                                                                                             */
    /*  00000c5046759d08-dmarc-request@listserv.uga.edu                                                                       */
    /*                                                                                                                        */
    /*  The problem is to find a maximum independent set.                                                                     */
    /*  As you noted, that is equivalent to finding a maximum clique in the complement.                                       */
    /*  PROC OPTNET will find (all or a fixed number of) maximal cliques,                                                     */
    /*  and you can access the same algorithm via the network solver in PROC OPTMODEL:                                        */
    /*                                                                                                                        */
    /*  So we see that there are 49 maximal cliques, and 3 of these are maximum cliques.                                      */
    /*  But finding all maximal cliques to find a maximum clique is generally inefficient.                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    proc optmodel;
       set <str> NODES;
       read data have into NODES=[node];
       num adjacency {NODES, NODES};
       read data have into [node] {j in NODES} <adjacency[node,j]=col(j)>;
       set EDGES = {i in NODES, j in NODES: i < j and adjacency[i,j] not in {.,0}};
       set COMPLEMENT = {i in NODES, j in NODES: i < j} diff EDGES;
       set <num,str> ID_NODE;
       solve with network / clique=(maxcliques=all) links=(include=COMPLEMENT) out=(cliques=ID_NODE);
       set CLIQUES init {};
       set <str> NODES_c {CLIQUES} init {};
       num cliqueSize {c in CLIQUES} = card(NODES_c[c]);
       for {<c,i> in ID_NODE} do;
          CLIQUES = CLIQUES union {c};
          NODES_c[c] = NODES_c[c] union {i};
       end;
       num maxCliqueSize = max {c in CLIQUES} cliqueSize[c];
       for {c in CLIQUES: cliqueSize[c] = maxCliqueSize} put NODES_c[c]=;
    quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
                                                                      
                                                                                                                             
     REPO                                                                                                                    
     ----------------------------------------------------------------------------------------------------------------------- 
     https://github.com/rogerjdeangelis/utl-R-AI-igraph-list-connections-in-a-non-directed-graph-for-a-subset-of-vertices    
     https://github.com/rogerjdeangelis/utl-how-many-triangles-in-the-polygon-r-igraph-AI                                    
     https://github.com/rogerjdeangelis/utl-identify-linked-and-unliked-paths-r-igraph                                       
     https://github.com/rogerjdeangelis/utl-shortest-and-longest-travel-time-from-home-to-work-igraph-AI                     
     https://github.com/rogerjdeangelis/utl_remove_isolated_nodes_from_an_network_r_igraph                                   

