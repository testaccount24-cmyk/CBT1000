/* Rexx ***********************************************************************/
/*                                                                            */
/*               MASSCHG - ISPF Edit Macro Mass Change Facility               */
/*                                                                            */
/* Author      :     DAN DIRKSE                                               */
/* Release     :     V4R2                                                     */
/* Last update :     11/01/20222                                              */
/* Parts Needed:                                                              */
/*      Panels :     MASSC001-MASSC005                                        */
/*                   MASSCT00, 10-13, 15, 20-26, 30                           */
/*      Msgs   :     MASSC10,MASSC11,MASSC12                                  */
/*      Skels  :     MASSKEL,MASSBAT1                                         */
/*      Macros :     $CHGIT, $FNDIT, $NOTFND, $LOGIT, $SETVVMM, MASSEXIT      */
/*                                                                            */
/******************************************************************************/
/*                                                                            */
/* This exec performs an ISPF Edit Macro against all members or a             */
/* selection of members of one to four partitioned data sets.                 */
/*                                                                            */
/* Editing can be performed in either the foreground or in batch              */
/*                                                                            */
/* See HELP Tutorial panels for more information.                             */
/*                                                                            */
/******************************************************************************/
/* Send questions, suggestions and/or bug reports to:                         */
/*                         Dan Dirkse                                         */
/*                   ztools.channel@gmail.com                                 */
/******************************************************************************/
/*                                                                            */
/*             (C) Copyright The Z Tools Company, 2021                        */
/*                                                                            */
/******************************************************************************/
/* This program is free software: you can redistribute it and/or modify it    */
/* under the terms of the GNU General Public License as published by the Free */
/* Software Foundation, either version 3 of the License, or (at your option)  */
/* any later version.                                                         */
/*                                                                            */
/* This program is distributed in the hope that it will be useful, but        */
/* WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY */
/* or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License    */
/* for more details.                                                          */
/*                                                                            */
/* You should have received a copy of the GNU General Public License along    */
/* with this program. If not, see https://www.gnu.org/licenses/               */
/******************************************************************************/
Arg trace .
If trace = "TRACE" Then Trace ?i

Address ISPEXEC "VGET (MASSEXEC MASSLIBS) SHARED"

Call Initialize
Call Main

Exit


/******************************************************************************/
/* Initialize variables                                                       */
/******************************************************************************/
Initialize:
/* OPTIONS BELOW */
viounit  = "VIO"
tempunit = "SYSDA"
skip. = 0 /* Set the number of dsns to skip for each ddname for batch job */
/* skip.ISPTLIB = 1 */
/* OPTIONS ABOVE */

massrlse = "V4R3"
Address ISPEXEC "CONTROL DISPLAY REFRESH"
done  = "N"
error = "N"
blank = " "
selected = "SELECTED"

Address ISPEXEC "VGET (ZUSER)"
Return


/******************************************************************************/
/* Main Processing Routine                                                    */
/******************************************************************************/
Main:
massstat = "NO"
csrfld = "MASSCDSN"
Do While done = "N"
  Address ISPEXEC " DISPLAY PANEL(MASSC001) CURSOR("csrfld")"
  mainrc = rc
  csrfld = "MASSCDSN"
  error = "N"

  If mainrc = 0 Then
    Do
      Call Set_unique
      Call Check_dsns
      If error = "N" & masscmac = "$FNDIT" Then
        Call Check_$FNDIT_parms
      If error = "N" & masscmac = "$NOTFND" Then
        Call Check_$NOTFND_parms
      If error = "N" & masscmac = "$CHGIT" Then
        Call Check_$CHGIT_parms
      If error = "N" & masscmac = "$LOGIT" Then
        Call Check_$LOGIT_parms
      If error = "N" & masscmac = "$SETVVMM" Then
        Call Check_$SETVVMM_parms

      If error = "N" Then
        Do
          retcd = 0
          If massmode = "B" Then                       /* Display Batch Panel */
            Do
              Parse Var blank MASSJOB1 MASSJOB2 MASSJOB3 MASSJOB4 ,
                              ZLLGJOB1 ZLLGJOB2 ZLLGJOB3 ZLLGJOB4
              Address ISPEXEC "VGET (MASSJOB1 MASSJOB2 MASSJOB3 MASSJOB4" ,
                                    "ZLLGJOB1 ZLLGJOB2 ZLLGJOB3 ZLLGJOB4)" ,
                                    "PROFILE"

              If massjob1 = blank & massjob2 = blank & ,
                 massjob3 = blank & massjob4 = blank Then
                Do
                  /* Default from LOG/LIST JCL */
                  massjob1 = zllgjob1
                  massjob2 = zllgjob2
                  massjob3 = zllgjob3
                  massjob4 = zllgjob4
                  Address ISPEXEC "SETMSG MSG(MASSC127)"
                End
              Address ISPEXEC " DISPLAY PANEL(MASSC003) CURSOR(MASSEDIT)"
              retcd = rc
            End

          If retcd = 0 Then
            Do
              If massstat = "YES" Then    /* Display Statistics Warning Panel */
                Do
                  Address ISPEXEC " DISPLAY PANEL(MASSC004) CURSOR(MASSSTAT)"
                  retcd = rc
                End
            End

          If retcd = 0 Then
            Do
              Call Allocate_dsns
              If error = "N" Then
                Call Initialize_ds

              If error = "N" Then
                Call Create_table

              If error = "N" Then
                Call Select_members

              If error = "N" & memcanc = "N" Then
                Do
                  Address ISPEXEC " TBSTATS "memtable" ROWCURR(MASSMEMT)"
                  If massmode = "F" Then
                    Call Process_foreground
                  Else If massmode = "B" Then
                    Call Process_background
                End

              If tbcreated = "Y" then
                Do
                  Address ISPEXEC "TBEND "memtable
                  Call Chkrc rc,"TBEND "memtable
                End
            End
        End
    End
  Else
    Done = "Y"
End

Exit


/******************************************************************************/
/* Check data set(s) for existence and verify they are partitioned data sets. */
/******************************************************************************/
Check_dsns:
  If masscdsn <> "" Then
    Do
      If Pos("(",masscdsn) > 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(MASSC117)"
          csrfld = "MASSCDSN"
          error = "Y"
        End
      Else If Sysdsn(masscdsn) = "OK" Then
        Do
          z = MSG("OFF")
          dsi_rc = listdsi(masscdsn)
          z = MSG("ON")
          If sysdsorg <> "PO" & sysdsorg <> "POU" Then
            Do
              Address ISPEXEC "SETMSG MSG(MASSC110)"
              csrfld = "MASSCDSN"
              error = "Y"
            End
        End
      Else
        Do
          Address ISPEXEC "SETMSG MSG(MASSC105)"
          csrfld = "MASSCDSN"
          error = "Y"
        End
    End

  If error = "N" & masscds2 <> "" Then
    Do
      If Pos("(",masscds2) > 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(MASSC117)"
          csrfld = "MASSCDS2"
          error = "Y"
        End
      Else If Sysdsn(masscds2) = "OK" Then
        Do
          z = MSG("OFF")
          dsi_rc = listdsi(masscds2)
          z = MSG("ON")
          If sysdsorg <> "PO" & sysdsorg <> "POU" Then
            Do
              Address ISPEXEC "SETMSG MSG(MASSC111)"
              csrfld = "MASSCDS2"
              error = "Y"
            End
        End
      Else
        Do
          Address ISPEXEC "SETMSG MSG(MASSC106)"
          csrfld = "MASSCDS2"
          error = "Y"
        End
    End

  If error = "N" & masscds3 <> "" Then
    Do
      If Pos("(",masscds3) > 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(MASSC117)"
          csrfld = "MASSCDS3"
          error = "Y"
        End
      Else If Sysdsn(masscds3) = "OK" Then
        Do
          z = MSG("OFF")
          dsi_rc = listdsi(masscds3)
          z = MSG("ON")
          If sysdsorg <> "PO" & sysdsorg <> "POU" Then
            Do
              Address ISPEXEC "SETMSG MSG(MASSC112)"
              csrfld = "MASSCDS3"
              error = "Y"
            End
        End
      Else
        Do
          Address ISPEXEC "SETMSG MSG(MASSC107)"
          csrfld = "MASSCDS3"
          error = "Y"
        End
    End

  If error = "N" & masscds4 <> "" Then
    Do
      If Pos("(",masscds4) > 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(MASSC117)"
          csrfld = "MASSCDS4"
          error = "Y"
        End
      Else If Sysdsn(masscds4) = "OK" Then
        Do
          z = MSG("OFF")
          dsi_rc = listdsi(masscds4)
          z = MSG("ON")
          If sysdsorg <> "PO" & sysdsorg <> "POU" Then
            Do
              Address ISPEXEC "SETMSG MSG(MASSC113)"
              csrfld = "MASSCDS4"
              error = "Y"
            End
        End
      Else
        Do
          Address ISPEXEC "SETMSG MSG(MASSC108)"
          csrfld = "MASSCDS4"
          error = "Y"
        End
    End
Return


/******************************************************************************/
/* Allocate data sets to edited                                               */
/******************************************************************************/
Allocate_dsns:
  ddinput = "D" || unique
  Address TSO "ALLOC F("ddinput") SHR REUSE" ,
              "DA("masscdsn ,
                   masscds2 ,
                   masscds3 ,
                   masscds4")"
  allocrc = rc
  If allocrc <> 0 Then  /* Error during allocation */
    Do
      Address ISPEXEC "SETMSG MSG(MASSC109)"
      error = "Y"
    End
Return


/******************************************************************************/
/* Check for $FNDIT parms                                                     */
/******************************************************************************/
Check_$FNDIT_parms:
If massparm = "" Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC121)"
    error = "Y"
    csrfld = "MASSPARM"
  End
Return


/******************************************************************************/
/* Check for $NOTFND parms                                                    */
/******************************************************************************/
Check_$NOTFND_parms:
If massparm = "" Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC128)"
    error = "Y"
    csrfld = "MASSPARM"
  End
Return


/******************************************************************************/
/* Check for $CHGIT parms                                                     */
/******************************************************************************/
Check_$CHGIT_parms:
If massparm <> "" Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC122)"
    error = "Y"
    csrfld = "MASSPARM"
  End
If (macparm1 = "" | macparm2 = "") & error = "N" Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC123)"
    error = "Y"
    If macparm1 = "" Then
      csrfld = "MACPARM1"
    Else
      csrfld = "MACPARM2"
  End
Return


/******************************************************************************/
/* Check for $LOGIT parms                                                     */
/******************************************************************************/
Check_$LOGIT_parms:
If massparm = "" Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC124)"
    error = "Y"
    csrfld = "MASSPARM"
  End
Return


/******************************************************************************/
/* Check for $SETVVMM parms                                                   */
/******************************************************************************/
Check_$SETVVMM_parms:
If massparm = "" Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC125)"
    error = "Y"
    csrfld = "MASSPARM"
  End
If Invalid_vvmm(massparm) Then
  Do
    Address ISPEXEC "SETMSG MSG(MASSC126)"
    error = "Y"
    csrfld = "MASSPARM"
  End
Return


/******************************************************************************/
/* Initialize and open data set(s) to be edited                               */
/******************************************************************************/
Initialize_ds:
  /* Initialize Data Set(s) */
  Address ISPEXEC " LMINIT DATAID(MASSID) DDNAME("ddinput")" ,
                  "ENQ(SHRW) ORG(MASSORG)"
  initrc = rc
  If initrc <> 0 Then   /* Data set not allocated */
    Do
      Address ISPEXEC "SETMSG MSG(ISRZ002)"
      error = "Y"
    End

  If error = "N" Then
    Do
      Address ISPEXEC "LMOPEN DATAID("massid") OPTION(INPUT)"
      openrc = rc
      If openrc <> 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(ISRZ002)"
          error = "Y"
        End
    End

Return


/******************************************************************************/
/* Initialize and open data set(s) for member selection                       */
/******************************************************************************/
Initialize_ds_mem:
  /* Initialize Data Set(s) */
  Address ISPEXEC " LMINIT DATAID(MASSIDM) DDNAME("ddinput")" ,
                  "ENQ(SHRW) ORG(MASSORG)"
  initmrc = rc
  If initmrc <> 0 Then   /* Data set not allocated */
    Do
      Address ISPEXEC "SETMSG MSG(ISRZ002)"
      error = "Y"
    End

  If error = "N" Then
    Do
      Address ISPEXEC "LMOPEN DATAID("massidm") OPTION(INPUT)"
      openmrc = rc
      If openmrc <> 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(ISRZ002)"
          error = "Y"
        End
    End

Return


/******************************************************************************/
/* Create table to store list of members                                      */
/******************************************************************************/
Create_table:
  memtable = "MC" || unique
  Address ISPEXEC " TBCREATE "memtable" KEYS(TBMEMBER)" ,
                  " NOWRITE REPLACE"
  tablerc = rc
  If tablerc <> 0 Then
    Do
      Address ISPEXEC "SETMSG MSG(ISRZ002)"
      error = "Y"
      tbcreated = "N"
    End
  Else
    tbcreated = "Y"
  Address ISPEXEC "TBSORT "memtable" FIELDS(TBMEMBER,C,A)"
  tbsortrc = rc
  If tbsortrc <> 0 Then
    Do
      Address ISPEXEC "SETMSG MSG(ISRZ002)"
      Say "TBSORT error" tbsortrc
      error = "Y"
    End
Return


/******************************************************************************/
/* Select members routine                                                     */
/******************************************************************************/
Select_members:
  disprc = 0
  Call Initialize_ds_mem
  zlmember = ""
  memdone = "N"
  memcanc = "N"
  Do While disprc < 8 & memdone = "N"

    Address ISPEXEC "LMMDISP DATAID("massidm") PANEL(MASSC005)" ,
         "OPTION(DISPLAY) MEMBER(&MASSCPAT)" ,
         "STATS(YES) TOP(&ZLMEMBER) COMMANDS(ANY)"

    disprc = rc
    getrc = disprc
    memcnt = 0
    firstmem = zlmember

    If zcmd = "CAN" | zcmd = "CANCEL" Then
      Do
        zcmd = ""
        Address ISPEXEC "SETMSG MSG(MASSC115)"
        memdone = "Y"
        memcanc = "Y"
      End
    Else If disprc = 4 Then
      Do
        /* No members in data set(s) or matching pattern */
        Address ISPEXEC "SETMSG MSG(MASSC100)"
        memdone = "Y"
      End
    Else
      Do While getrc < 8
        /* Check if already selected, set prevdata */
        tbmember = zlmember
        Address ISPEXEC "TBEXIST "memtable
        existrc = rc
        If existrc = 0 Then
          prevdata = selected
        Else
          prevdata = blank

        If zllcmd = "S" Then
          Do
            If existrc <> 0 Then
              Do
                Address ISPEXEC "LMMDISP DATAID("massidm")" ,
                       "OPTION(PUT) MEMBER("zlmember")" ,
                       "ZLLCMD("blank") ZLUDATA("selected")"
                tbmember = zlmember
                Address ISPEXEC "TBADD "memtable" ORDER"
                tbrc = rc
                memcnt = memcnt + 1
              End
            memfunc = "selected"
          End
        Else If zllcmd = "X" Then
          Do
            If existrc = 0 Then
              Do
                Address ISPEXEC "LMMDISP DATAID("massidm")" ,
                       "OPTION(PUT) MEMBER(&ZLMEMBER)" ,
                       "ZLLCMD(&BLANK) ZLUDATA(&BLANK)"
                Address ISPEXEC "LMMDISP DATAID("massidm")" ,
                       "OPTION(PUT) MEMBER("zlmember")" ,
                       "ZLLCMD("blank") ZLUDATA("blank")"
                tbmember = zlmember
                Address ISPEXEC "TBTOP "memtable
                Call Chkrc rc,"TBTOP "memtable
                Address ISPEXEC "TBSCAN "memtable" ARGLIST(TBMEMBER)"
                Call Chkrc rc,"TBSCAN "memtable" ARGLIST(TBMEMBER)"
                Address ISPEXEC "TBDELETE "memtable
                Call Chkrc rc,"TBDELETE "memtable
                memcnt = memcnt + 1
              End
            memfunc = "excluded"
          End
        Else If zllcmd = "B" Then
          Do
            Address ISPEXEC "BROWSE DATAID("massidm") MEMBER("ZLMEMBER")"
            Address ISPEXEC "LMMDISP DATAID("massidm")" ,
                   "OPTION(PUT) MEMBER("zlmember")" ,
                   "ZLLCMD("blank") ZLUDATA("prevdata")"
            memcnt = memcnt + 1
            memfunc = "browsed"
          END
        Else If zllcmd = "V" Then
          Do
            Address ISPEXEC "VIEW DATAID("massidm") MEMBER("ZLMEMBER")"
            Address ISPEXEC "LMMDISP DATAID("massidm")" ,
                   "OPTION(PUT) MEMBER("zlmember")" ,
                   "ZLLCMD("blank") ZLUDATA("prevdata")"
            memcnt = memcnt + 1
            memfunc = "viewed"
          End
        Else If zllcmd = "E" Then
          Do
            Address ISPEXEC "EDIT DATAID("massidm") MEMBER("ZLMEMBER")"
            editrc = rc
            If massstat = "YES" & editrc = 0 Then
            Address ISPEXEC "LMMSTATS DATAID("massidm")" ,
             "MEMBER("zlmember")" ,
             "CREATED("zlcdate") MODDATE("zlmdate")" ,
             "MODTIME("zlmtime":"zlmsec") INITSIZE("zlinorc")" ,
             "MODRECS("zlmnorc") USER("zluser")"
            Address ISPEXEC "LMMDISP DATAID("massidm")" ,
                   "OPTION(PUT) MEMBER("zlmember")" ,
                   "ZLLCMD("blank") ZLUDATA("prevdata")"
            memcnt = memcnt + 1
            memfunc = "edited"
          End
        Else
          Do
            Address ISPEXEC "SETMSG MSG(MASSC102)"
          End
        Address ISPEXEC "LMMDISP DATAID("massidm")" ,
               "OPTION(GET) STATS(YES)"
        getrc = rc
      End
    If memcnt = 1 Then
      Address ISPEXEC "SETMSG MSG(MASSC118)"
    Else
      Address ISPEXEC "SETMSG MSG(MASSC119)"
    zlmember = firstmem
  End
 If openmrc = 0 Then
   Do
     Address ISPEXEC "LMCLOSE DATAID("massidm")"
     Call Chkrc rc,"LMCLOSE DATAID("massidm")"
   End
  If initmrc = 0 Then
    Do
      Address ISPEXEC "LMFREE DATAID("massidm")"
      Call Chkrc rc,"LMFREE DATAID("massidm")"
    End
  If memcanc = "N" Then
    Do
      Address ISPEXEC "TBQUERY "memtable" ROWNUM(TABLROWS)"
      If tablrows = 0 Then
        Do
          Address ISPEXEC "SETMSG MSG(MASSC114)"
          error = "Y"
        End
    End
Return


/******************************************************************************/
/* Perform foreground processing                                              */
/******************************************************************************/
Process_foreground:
  Address ISPEXEC "VPUT (MACPARM1 MACPARM2 MACPARM3 MACPARM4) SHARED"
  massmem# = 0
  proccnt = 0
  massexit = ""
  Address ISPEXEC "VPUT (MASSEXIT) SHARED"

  Address ISPEXEC "TBTOP "memtable
  Address ISPEXEC "TBSKIP "memtable
  skiprc = rc
  Do While skiprc < 8 & massexit = ""
    massmem# = massmem# + 1
    massmemf = "NO"
    massmeml = "NO"
    If massmem# = 1 Then
      massmemf = "YES"
    IF massmem# = massmemt Then
      massmeml = "YES"
    Address ISPEXEC "VPUT (MASSMEMF,MASSMEML,MASSMEM#,MASSMEMT) SHARED"

    Address ISPEXEC "CONTROL DISPLAY LOCK"
    ipmsg = "Processing Member "tbmember
    Address ISPEXEC "DISPLAY PANEL(MASSC002)"
    If massstat = "YES" Then
      Address ISPEXEC "LMMFIND DATAID("massid") MEMBER("tbmember")" ,
              "STATS(YES)"

    /* If parm passed to a macro whose MACRO statement does not contain a     */
    /* parm field, e.g. MACRO(PARMIN), the statement will receive a return    */
    /* code of 8.                                                             */
    If massparm <> "" Then
      editparm = "PARM(MASSPARM)"
    Else
      editparm = ""
    Address ISPEXEC "EDIT DATAID("massid") MEMBER("tbmember")" ,
            "MACRO("masscmac") CONFIRM(NO)" editparm
    editrc = rc

    If massstat = "YES" & editrc = 0 Then
      Address ISPEXEC "LMMSTATS DATAID("massid") MEMBER("tbmember")" ,
       "CREATED("zlcdate") MODDATE("zlmdate")" ,
       "MODTIME("zlmtime":"zlmsec") INITSIZE("zlinorc")" ,
       "MODRECS("zlmnorc") USER("zluser")"
    proccnt = proccnt + 1

    Address ISPEXEC "VGET (MASSEXIT) SHARED"
    If massexit = "" Then
      Do
        Address ISPEXEC "TBSKIP "memtable
        skiprc = rc
      End
  End
  /* Set number of processed message */
  If massexit = "" Then
    Address ISPEXEC "SETMSG MSG(MASSC101) COND"
  Else
    Address ISPEXEC "SETMSG MSG(MASSC103) COND"
 If openrc = 0 Then
   Do
     Address ISPEXEC "LMCLOSE DATAID("massid")"
     Call Chkrc rc,"LMCLOSE DATAID("massid")"
   End

 If initrc = 0 Then
   Do
     Address ISPEXEC "LMFREE DATAID("massid")"
     Call Chkrc rc,"LMFREE DATAID("massid")"
   End
 If allocrc = 0 Then
   Do
     Address TSO "FREE FI("ddinput")"
     Call Chkrc rc,"FREE FI("ddinput")"
   End
Return


/******************************************************************************/
/* Perform background processing                                              */
/******************************************************************************/
Process_background:
  numrows = massmemt
  Address ISPEXEC "VPUT (NUMROWS) SHARED"
  backid = "B" || unique

  Address ISPEXEC "CONTROL DISPLAY LOCK"
  ipmsg = "Writing List of Members"
  Address ISPEXEC "DISPLAY PANEL(MASSC002)"

  /* Allocate data set for generated exec    */
  execdsn = zuser".MASSCHG."backid".EXEC"
  execdd = "EX" || unique
  Address TSO "ALLOC NEW CAT REU DD("execdd") DA('"execdsn"')" ,
    "LRECL(72) RECFM(F B)" ,
    "DSORG(PO) TRACK UNIT("tempunit") SP(4,4)" ,
    "DIR(10)"
  Address TSO "FREE FI("execdd")"

  /* Allocate data set for list of members   */
  memdsn = zuser".MASSCHG."backid".MEMBERS"
  memdd  = "MB" || unique
  Address TSO "ALLOC NEW CAT REU DD("memdd") DA('"memdsn"')" ,
    "LRECL(20) BLKSIZE(6160)        RECFM(F B)" ,
    "DSORG(PS) TRACK UNIT("tempunit") SP(20,5)"

  Address TSO "EXECIO 0 diskw "memdd" (open"

  Address ISPEXEC "TBTOP "memtable
  Address ISPEXEC "TBSKIP "memtable
  skiprc = rc
  Do While skiprc < 8
    mem.1 = tbmember
    Address TSO "EXECIO 1 diskw "memdd" (stem mem."
    Address ISPEXEC "TBSKIP "memtable
    skiprc = rc
  End
  Address TSO "EXECIO 0 diskw "memdd" (finis"
  Address TSO "FREE FI("memdd")"

  ddtable = "DD" || unique
  Address ISPEXEC "TBCREATE "ddtable" NAMES(TDDNAME TDSNAME)" ,
                  "NOWRITE REPLACE"
  tsysprc=rc

  xddlist = "ISPPROF SYSPROC SYSEXEC ISPPLIB ISPMLIB ISPSLIB" ,
            "ISPLLIB ISPTLIB"
  Call DD_stems(xddlist)
  ispprof_dsn = ispprof.dsn.1

  Do xdd = 1 To Words(xddlist)
    tddname = Word(xddlist,xdd)
    currdd  = tddname
    Interpret "dsncnt = "tddname".dsn.0"
    Do dddsn = skip.tddname + 1 To dsncnt
      Interpret "tdsname = "currdd".dsn."dddsn
      If (currdd = "ISPPROF" | ,
         (currdd = "ISPTLIB" & tdsname = ispprof_dsn)) Then
        Nop
      Else
        Do
          Address ISPEXEC "TBADD "ddtable" ORDER"
          tddname = Copies(" ",Length(tddname)) /* Clear DD name */
        End
    End
  End

  Address ISPEXEC "TBTOP "ddtable
  Address ISPEXEC "CONTROL DISPLAY LOCK"
  ipmsg = "Creating Exec to be run in Batch"
  Address ISPEXEC "DISPLAY PANEL(MASSC002)"

  Address TSO "ALLOC FI(ISPFILE) DA('"execdsn"') SHR REUSE"
  Call Chkrc rc,"ALLOC FI(ISPFILE)"
  Address ISPEXEC "FTOPEN"
  Call Chkrc rc,"FTOPEN"
  Address ISPEXEC "FTINCL MASSBAT1"
  Call Chkrc rc,"FTINCL MASSBAT1"
  Address ISPEXEC "FTCLOSE NAME(MASSBAT1)"
  Call Chkrc rc,"FTCLOSE MASSBAT1"
  Address ISPEXEC "CONTROL DISPLAY LOCK"
  ipmsg = "Creating JCL for Batch Job"
  Address ISPEXEC "DISPLAY PANEL(MASSC002)"

  Address ISPEXEC "FTOPEN TEMP"
  Call Chkrc rc,"FTOPEN"
  Address ISPEXEC "FTINCL MASSKEL"
  Call Chkrc rc,"FTINCL MASSSKEL"
  Address ISPEXEC "FTCLOSE"
  Call Chkrc rc,"FTCLOSE MASSSKEL"

  Address ISPEXEC "VGET (ZTEMPF,ZTEMPN)"
  Address TSO "FREE FI(ISPFILE)"
  If massedit = "YES" Then
    Do
      Address ISPEXEC "LMINIT DATAID(EDDID) DDNAME("ztempn") ENQ(EXCLU)"
      Address ISPEXEC "EDIT DATAID(&EDDID)"
      Address ISPEXEC "LMFREE DATAID(&EDDID)"
    End
  Else
    Address TSO "SUBMIT '"ztempf"'"
  If (tsysprc = 0 | tsysprc = 4) Then
    Address ISPEXEC "TBEND "ddtable
  If openrc = 0 Then
    Do
      Address ISPEXEC "LMCLOSE DATAID("massid")"
      Call Chkrc rc,"LMCLOSE DATAID("massid")"
    End
  If initrc = 0 Then
    Do
      Address ISPEXEC "LMFREE DATAID("massid")"
      Call Chkrc rc,"LMFREE DATAID("massid")"
    End
  If allocrc = 0 Then
    Do
      Address TSO "FREE FI("ddinput")"
      Call Chkrc rc,"FREE FI("ddinput")"
    End

Return

/******************************************************************************/
/* Stores DSN Allocations in stem variables with ddname as first part of the  */
/* stems.  For example, SYSPROC.DSN.1, SYSPROC.DISP.1                         */
/******************************************************************************/
DD_Stems:
Arg xddlist
Do xdd = 1 To Words(xddlist)
  currdd = Word(xddlist,xdd)
  Interpret currdd".dsn.0 = 0"
End
q = Outtrap(trap.,3000,"CONCAT")
Address TSO "LISTA ST"
q = Outtrap("OFF")
store_stems = "N"
save_xddname = ""
xseq. = 0
Do xi = 2 To trap.0
  Parse Var trap.xi first2 +2 3 xddname +8 12 xdisp 1 xdsn +44
  xddname = Strip(xddname,"T")
  If first2 <> "  " Then /* DSNAME */
    Do
      save_xdsn = xdsn
    End
  Else                   /* DDNAME or blank */
    Do
      If xddname <> "" Then /* DDNAME */
        Do
          If WordPos(xddname,xddlist) > 0 Then
            Do
              store_stems = "Y"
              save_xddname = xddname
              xseq.xddname = 0
            End
          Else
            store_stems = "N"
        End
      Else If store_stems = "Y" Then   /* Same DDNAME */
        Do
          xddname = save_xddname
        End
      If store_stems = "Y" Then
        Do
          xseq.xddname = xseq.xddname + 1
          currxseq     = xseq.xddname
          Interpret xddname".dsn."currxseq" = "save_xdsn
          Interpret xddname".dsn.0 = "currxseq
          Interpret xddname".disp."currxseq" = "xdisp
          Interpret xddname".disp.0 = "currxseq
        End
    End
End
Return


/******************************************************************************/
/* Check RC for funtions and error if not 0                                   */
/******************************************************************************/
Chkrc: Procedure
Arg rcin, function
If rcin /= 0 Then
  Do
    Say "MASSBAT1 Exec - "function" Issued Return Code "rcin
    zispfrc = 8
    Address ISPEXEC "VPUT (ZISPFRC) SHARED"
    Exit 8
  End
Return


/******************************************************************************/
/* Set a six digit unique id for use as a suffix for various things           */
/******************************************************************************/
Set_unique: Procedure Expose unique
day       = Substr(Date("U"),4,2)
min       = Substr(Time(),4,2)
sec       = Substr(Time(),7,2)
unique = day || min || sec
Return


/******************************************************************************/
/* Set 1 if parm is invalid                                                   */
/******************************************************************************/
Invalid_vvmm: Procedure Expose vv mm
Arg parmin
invalid = 0
If Left(parmin,1) = "." Then
  Do
    vv = ""
    mm = Substr(parmin,2)
  End
Else If Pos(".",parmin) = 0 Then
  Do
    vv = parmin
    mm = ""
  End
Else
  Parse Var parmin vv "." mm
If (vv = "" & mm = "") | Length(vv) > 2 | Length(mm) > 2 | ,
    Pos(".",mm > 0)    | (Datatype(vv,"W") = 0 & vv <> "") | ,
    (Datatype(mm,"W") = 0 & mm <> "") Then
  invalid = 1
Return invalid
