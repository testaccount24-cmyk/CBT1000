/*REXX*****************************************************************/
/* Initial edit macro used to perform a change all command for the    */
/* given strings supplied in the MACPARM1 and MACPARM2 variables.     */
/* Optionally, MACPARM3 can also be supplied.                         */
/*                                                                    */
/* This macro is for use in conjunction with the MASSCHG exec.        */
/*                                                                    */
/* This macro contains an ISREDIT END command so it will make the     */
/* change without displaying the editor.                              */
/**********************************************************************/
Address ISREDIT "MACRO"
mode = Sysvar('SYSENV')                        /* Running in FORE or BACK     */
Address ISPEXEC "VGET (MACPARM1,MACPARM2,MACPARM3) SHARED"
If macparm1 = '' | macparm2 = "" Then
  Do
    zedsmsg = "Missing Parameter(s)"
    zedlmsg = "The $CHGIT macro requires MACPARM1 and MACPARM2 values"
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
Address ISPEXEC "CONTROL ERRORS RETURN"        /* Capture change command error*/

Address ISREDIT "CHANGE ALL "macparm1 macparm2 macparm3
Call Edit_error

Address ISREDIT "USER_STATE = (US)"            /* Restore current user state  */
Address ISREDIT "END"
Call Edit_error
Address ISPEXEC "CONTROL ERRORS CANCEL"        /* Normal error processing     */
Exit

Edit_error:
If rc > 4 Then
  Do
    If mode = "FORE" Then
      Address ISPEXEC "SETMSG MSG(ISRZ002)"    /* Display ISPFs error         */
    Else
      Say zerrsm "-" zerrlm
    Address ISREDIT "CANCEL"
    massexit = "YES"
    Address ISPEXEC "VPUT (MASSEXIT) SHARED"   /* Set MASSEXIT variable       */
  End
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
