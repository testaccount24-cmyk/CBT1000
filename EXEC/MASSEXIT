/*REXX*****************************************************************/
/* This macro is for use in conjunction with the MASSCHG exec.        */
/*                                                                    */
/* The macro sets a variable named MASSEXIT to YES and stores it into */
/* the Shared pool to interrupt processing of the members.  It is     */
/* useful if you forget to include an ISREDIT END or CANCEL in your   */
/* macro and you are processing a large list in the foreground        */
/* or you experience a macro error.                                   */
/**********************************************************************/
Address ISREDIT "MACRO"
massexit = "YES"
Address ISPEXEC "VPUT (MASSEXIT) SHARED"
zedsmsg = "MASSCHG will exit"
zedlmsg = "MASSCHG will exit after you leave this member"
Address ISPEXEC "SETMSG MSG(ISRZ000)"
Exit
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
