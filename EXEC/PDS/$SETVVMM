/*REXX*************************************************************************/
/* Initial edit macro used to set version and/or level, save, and end.        */
/*   Parameter format:                                                        */
/*      2.3       Set Version to 02 and Level to 03                           */
/*      4         Set Version to 04                                           */
/*      .12       Set Level to 12                                             */
/*                                                                            */
/* This macro is for use in conjunction with the MASSCHG exec.                */
/*                                                                            */
/* If the parm is missing or an invalid format then MASSEXIT will be set to   */
/* YES and VPUT to the Shared pool.                                           */
/******************************************************************************/
Address ISREDIT "MACRO (VVMM)"
mode = Sysvar('SYSENV')                        /* Running in FORE or BACK     */
If vvmm = "" Then
  Do
    zedsmsg = "Missing Parameter"
    zedlmsg = "The $SETVVMM macro requires a parameter: vv.mm to set" ,
              "the version and level, vv to set version only, .mm to" ,
              "set only the level where vv = 01-99 and mm = 00-99."
    If mode = "FORE" Then
      Address ISPEXEC "SETMSG MSG(ISRZ001)"
    Else
      Say zedsmsg "-" zedlmsg
    massexit = "YES"
    Address ISPEXEC "VPUT (MASSEXIT) SHARED"   /* Set MASSEXIT variable       */
    Address ISREDIT "CANCEL"
    Exit
  End
Else If Invalid_vvmm(vvmm) Then
  Do
    zedsmsg = "Invalid Parameter"
    zedlmsg = "The $SETVVMM macro requires a parameter: vv.mm to set" ,
              "the version and level, vv to set version only, .mm to" ,
              "set only the level where vv = 01-99 and mm = 00-99."
    If mode = "FORE" Then
      Address ISPEXEC "SETMSG MSG(ISRZ001)"
    Else
      Say zedsmsg "-" zedlmsg
    massexit = "YES"
    Address ISPEXEC "VPUT (MASSEXIT) SHARED"   /* Set MASSEXIT variable       */
    Address ISREDIT "CANCEL"
    Exit
  End

Address ISPEXEC "CONTROL ERRORS RETURN"        /* Capture find command error  */
If vv <> "" Then
  Address ISREDIT "VERSION "vv
If rc > 4 Then
  Call Process_error

If mm <> "" Then
  Address ISREDIT "LEVEL" mm
If rc > 4 Then
  Call Process_error

Address ISREDIT "END"
If rc > 4 Then
  Call Process_error

Exit

/******************************************************************************/
/* Process find error and set MASSEXIT to YES                                 */
/******************************************************************************/
Process_error:
If mode = "FORE" Then
  Address ISPEXEC "SETMSG MSG(ISRZ002)"    /* Display ISPFs error         */
Else
  Say zerrsm "-" zerrlm
massexit = "YES"
Address ISPEXEC "VPUT (MASSEXIT) SHARED"   /* Set MASSEXIT variable       */
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
