/*REXX*************************************************************************/
/* Example macro included with the MASSCHG package to demonstrate the use of  */
/* the MASSMEMF (first member) and MASSMEML (last member) variables.          */
/*                                                                            */
/* This macro is for use in conjunction with the MASSCHG exec.                */
/* If run in the foreground, the data set will be viewed after the last       */
/* member.                                                                    */
/*                                                                            */
/* Because $LOGIT is only being used to gather information and not make any   */
/* changes, the macro contains an ISREDIT CANCEL command.  If you use this    */
/* as the basis for other macros that may also make changes, you will want to */
/* instead use the ISREDIT END command to save any changes.                   */
/*                                                                            */
/* An alternative to using EXECIO is to use the ISPEXEC LIST service.  In     */
/* this case, ISPF pre-allocates/opens the data set and you can use the LIST  */
/* command to save the data set.                                              */
/******************************************************************************/
Address ISREDIT "MACRO (FINDSTR)"
mode = Sysvar('SYSENV')                        /* Running in FORE or BACK     */
If findstr = "" Then
  Do
    zedsmsg = "Missing Parameter"
    zedlmsg = "The $LOGIT macro requires a valid find string specified" ,
              "in the Macro parameter field on the MASSCHG panel"
    If mode = "FORE" Then
      Address ISPEXEC "SETMSG MSG(ISRZ001)"
    Else
      Say zedsmsg "-" zedlmsg
    massexit = "YES"
    Address ISPEXEC "VPUT (MASSEXIT) SHARED"   /* Set MASSEXIT variable       */
    Address ISREDIT "CANCEL"
    Exit
  End

Address ISPEXEC "VGET (ZUSER)"
mass$log = "'"zuser".$LOGIT.CSV'"                 /* name of output data set  */

Address ISPEXEC "VGET (MASSMEMF,MASSMEML) SHARED"

If massmemf = "YES" Then                          /* First member?            */
  Call Allocate_open_headings

Address ISREDIT "(US) = USER_STATE"            /* Save current user state     */
Address ISREDIT "BOUNDS"                       /* Set bounds to full range    */
Address ISREDIT "(MEM) = MEMBER"               /* Save the member name        */
Address ISPEXEC "CONTROL ERRORS RETURN"        /* Capture find command error  */

/* Gather information - in this case, count the number of lines, the number   */
/* of strings found and the number lines they were found on.                  */
Address ISREDIT "(totallns) = LINENUM .ZLAST"     /* Find count of lines      */
totallns = totallns + 0                           /* Remove leading zeros     */
Address ISREDIT "FIND "findstr" ALL"
findrc = rc
If findrc > 4 Then
  Call Process_error
Else
  Do
    Address ISREDIT "(found,foundlns) = FIND_COUNTS"  /* Get find counts      */
    found = found + 0                                 /* Remove leading zeros */
    foundlns = foundlns + 0                           /* Remove leading zeros */
    Call Write_info
  End

Address ISPEXEC "CONTROL ERRORS CANCEL"        /* Normal error processing     */
Address ISREDIT "USER_STATE = (US)"            /* Restore current user state  */
If massmeml = "YES" Then                          /* Last member?             */
  Call Close_and_free

Address ISREDIT "CANCEL"                          /* Cancel                   */
Exit


/******************************************************************************/
/* Delete, Allocate, Open and write headings to output data set               */
/******************************************************************************/
Allocate_open_headings:

If Sysdsn(mass$log) = "OK" Then
  Do
    xxx = msg("OFF")
    Address TSO "DELETE "mass$log
    xxx = msg("ON")
  End
Address TSO "ALLOC F(INFOOUT) DA("mass$log") NEW MOD REUSE" ,
            "DSORG(PS) TRACKS SPACE(20,20)" ,
            "RECFM(F B) LRECL(80)"
Address TSO" EXECIO 0 DISKW INFOOUT (OPEN"

Address ISPEXEC "VPUT (MASS$LOG) SHARED"
Call Write_heading

Return


/******************************************************************************/
/* Format and write heading to outpuot data set                               */
/******************************************************************************/
Write_heading:
                      ,
info.1 = "Member,Total lines,Found chars,Found lines"
Address TSO" EXECIO 1 DISKW INFOOUT (STEM info."
Return


/******************************************************************************/
/* Format and write information to output data set                            */
/******************************************************************************/
Write_info:

info.1 = Left(mem,8)","totallns","found","foundlns
Address TSO" EXECIO 1 DISKW INFOOUT (STEM info."

Return


/******************************************************************************/
/* Close and free output data set                                             */
/******************************************************************************/
Close_and_free:

Address TSO" EXECIO 0 DISKW INFOOUT (FINIS"
Address TSO "FREE F(INFOOUT)"

If mode = "FORE" Then
  Address ISPEXEC "VIEW DATASET("mass$log")"

Return


/******************************************************************************/
/* Process find error and set MASSEXIT to YES                                 */
/******************************************************************************/
Process_error:
If mode = "FORE" Then
  Address ISPEXEC "SETMSG MSG(ISRZ002)"    /* Display ISPFs error         */
Else
  Say zerrsm "-" zerrlm
Address TSO" EXECIO 0 DISKW INFOOUT (FINIS"
Address TSO "FREE F(INFOOUT)"

massexit = "YES"
Address ISPEXEC "VPUT (MASSEXIT) SHARED"   /* Set MASSEXIT variable       */
Return
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
