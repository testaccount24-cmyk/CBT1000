/* Rexx */
/*Name: @MASSCHG */
Parse Source .  .  execmem  . massexec . . ispf .
/* Data sets expected to match the first n qualifiers of massexec */
masslibs = "ISPPLIB.PANELS ISPMLIB.MSGS ISPSLIB.SKELS"
dsnparts = Translate(massexec," ",".")
dsnprefix = Translate(Subword(dsnparts,1,Words(dsnparts)-1),"."," ")
memtorun = Substr(execmem,2)
Address ISPEXEC "VPUT (MASSEXEC MASSLIBS) SHARED"

Trace
x = Listdsi("'"massexec"'")
If x <> 0 Then
  Do
    Say "Could not locate DSN="massexec
    Exit 8
  End

x = Listdsi("'"massexec"("memtorun")'")
If x <> 0 Then
  Do
    Say "Could not locate DSN="massexec
    Exit 8
  End

If ispf <> "ISPF" Then
  Do
    Say execmem" is an ISPF application, run under ISPF"
    Exit 8
  End

Address TSO "ALTLIB ACTIVATE APPLICATION(EXEC) DA('"massexec"')"

Do l = 1 to Words(masslibs)
  lib = Word(masslibs,l)
  Parse Var lib dd "." qual
  libdsn = dsnprefix"."qual
  Address ISPEXEC "LIBDEF" dd "DATASET ID('"libdsn"')"
End

Address ISPEXEC "SELECT CMD(%"memtorun")"

Address TSO "ALTLIB DEACTIVATE APPLICATION(EXEC)"
Do l = 1 to Words(masslibs)
  lib = Word(masslibs,l)
  Parse Var lib dd "." qual
  libdsn = dsnprefix"."qual
  Address ISPEXEC "LIBDEF" dd "DATASET"
End

Exit
