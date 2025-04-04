/*REXX*************************************************************************/
/* Initial edit macro used to perform a find for the find string and any      */
/* any optional parameters passed.                                            */
/*                                                                            */
/* This macro is for use in conjunction with the MASSCHG exec.                */
/*                                                                            */
/* If the string is found, the macro will issue a CANCEL command so the       */
/* member will not be displayed and MASSCHG will process the next member.     */
/* If the string is not found and the macro is running in the foreground, the */
/* macro will exit leaving you in the editor to perform any needed updates.   */
/* Enter END or CANCEL to continue with the other selected members.           */
/* If the string is not found and the macro is running in the background, the */
/* macro will Say the member, line number and data for each occurrence        */
/* found and then issue a CANCEL command.                                     */
/* If the string is invalid, MASSEXIT will be set to YES and VPUT to the      */
/* Shared pool.                                                               */
/******************************************************************************/
Address ISREDIT "MACRO (FINDSTR)"
mode = Sysvar('SYSENV')                        /* Running in FORE or BACK     */
If findstr = "" Then
  Do
    zedsmsg = "Missing Parameter"
    zedlmsg = "The $FNDIT macro requires a valid find string specified" ,
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

Address ISREDIT "(US) = USER_STATE"            /* Save current user state     */
Address ISREDIT "BOUNDS"                       /* Set bounds to full range    */
Address ISREDIT "(MEM) = MEMBER"               /* Save the member name        */
Address ISPEXEC "CONTROL ERRORS RETURN"        /* Capture find command error  */

Address ISREDIT "FIND "findstr
findrc = rc
If findrc > 4 Then
  Call Process_error
Else If findrc = 4 & mode = "BACK" Then
  Do
    Address ISREDIT "(lastrw,lastcl) = CURSOR" /* Save cursor                 */
    Call Write_info
  End
Else If findrc = 4 & mode = "FORE" Then
  Do
    zedsmsg = ""
    zedlmsg = "FIND" findstr "was unsuccessful"
    Address ISPEXEC "SETMSG MSG(ISRZ001)"
  End

Address ISPEXEC "CONTROL ERRORS CANCEL"        /* Normal error processing     */
Address ISREDIT "USER_STATE = (US)"            /* Restore current user state  */
If findrc = 0 | mode = "BACK" Then
  Address ISREDIT "CANCEL"                     /* Cancel from the edit session*/
Exit

/******************************************************************************/
/* Write information to output                                                */
/******************************************************************************/
Write_info:
Address ISREDIT "(CURRLINE) = LINENUM .ZCSR"
currline = Right(currline+0,8)
Address ISREDIT "(CURRDATA) = LINE .ZCSR"
Say Left(mem,9)"FIND" findstr "was unsuccessful"
Return

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
/* Send questions, suggestions and/or bug reports to:                         */
/*                         Dan Dirkse                                         */
/*                   ztools.channel@gmail.com                                 */
/******************************************************************************/
/*                                                                            */
/*             (C) Copyright The Z Tools Company, 2022                        */
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
