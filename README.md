# CBT441
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 441 is from Brian Vogt of EDS, and contains 2 programs    *   FILE 441
//*           for RACF:                                             *   FILE 441
//*                                                                 *   FILE 441
//*           (1) RESUME - to recover from a situation in which all *   FILE 441
//*               userids with SPECIAL or OPERATIONS attributes     *   FILE 441
//*               have been revoked.  The program runs as a         *   FILE 441
//*               started task, and mimics the effect of a          *   FILE 441
//*               "TSO ALU userid RESUME" command.                  *   FILE 441
//*           (2) RA#NAMES - list all userids & user's names to a   *   FILE 441
//*               data set, and all group ids & installation-data   *   FILE 441
//*               to another data set - (all one-line entries).     *   FILE 441
//*                                                                 *   FILE 441
//*           email: "Vogt, Brian A" <brian.vogt@eds.com>           *   FILE 441
//*                                                                 *   FILE 441
//*     RESUME                                                      *   FILE 441
//*     ======                                                      *   FILE 441
//*       DATE   - November 1987.    MVS/SP 2.1.7 with RACF 1.7.0.  *   FILE 441
//*        SMF logging and updating of last access added July       *   FILE 441
//*        1988.  Tested ok on MVS 5.2.2 with RACF 2.2 and          *   FILE 441
//*        also on OS/390 2.6 (Jan 2000).                           *   FILE 441
//*                                                                 *   FILE 441
//*       FUNCTION -                                                *   FILE 441
//*        Use ICHEINTY to modify the following in a RACF USER      *   FILE 441
//*        profile :                                                *   FILE 441
//*         (1) unset FLAG4 (REVOKE attribute).                     *   FILE 441
//*         (2) zero out REVOKECT (revoke count).                   *   FILE 441
//*         (3) set LJTIME & LJDATE to the current time & date.     *   FILE 441
//*         (4) For the benefit of the RACF Auditor, write a type   *   FILE 441
//*             80 SMF record (pretending to be ALTUSER with the    *   FILE 441
//*             RESUME parameter).                                  *   FILE 441
//*                                                                 *   FILE 441
//*        The most likely practical applications are :             *   FILE 441
//*         (a) A hacker revokes all of the privileged userids in   *   FILE 441
//*             the system, by submitting many batch jobs with      *   FILE 441
//*             incorrect passwords.  (Never trust an Operator to   *   FILE 441
//*             reply correctly to message ICH302D).  This program  *   FILE 441
//*             cannot be thwarted, as it does not run under a      *   FILE 441
//*             RACF userid.                                        *   FILE 441
//*                                                                 *   FILE 441
//*         (b) The userid of a production batch job becomes        *   FILE 441
//*             revoked overnight.  The MVS Operator can overcome   *   FILE 441
//*             this, with a bit of information from the on-call    *   FILE 441
//*             Security Admin.                                     *   FILE 441
//*                                                                 *   FILE 441
//*       SETUP DOCUMENTATION -                                     *   FILE 441
//*       -------------------                                       *   FILE 441
//*        (1) This program must be invoked from a started task.    *   FILE 441
//*              //RESUME   PROC U=,PW=                             *   FILE 441
//*              //RESUME   EXEC PGM=RESUME,PARM='&USER,&PW'        *   FILE 441
//*        (2) The started task name must be RESUME.                *   FILE 441
//*        (3) Do NOT put the started task name into the RACF       *   FILE 441
//*            Started Procedures Table (ICHRIN03) or create a      *   FILE 441
//*            STARTED profile for it.  The task doesn't need a     *   FILE 441
//*            userid, and is better off without one.               *   FILE 441
//*                                                                 *   FILE 441
//*       USER DOCUMENTATION -                                      *   FILE 441
//*       ------------------                                        *   FILE 441
//*        (1) The function is invoked via this MVS command:        *   FILE 441
//*              S RESUME,U=userid,PW=password                      *   FILE 441
//*        (2) The userid (U keyword) must be specified.            *   FILE 441
//*        (3) The password (PW keyword) must match the RVARY       *   FILE 441
//*            SWITCH password.  If there is no RVARY SWITCH        *   FILE 441
//*            password (RCVTSWPW is binary zeroes), this parameter *   FILE 441
//*            is ignored, and may be entirely omitted from the     *   FILE 441
//*            START command.                                       *   FILE 441
//*        (4) The RVARY SWITCH password should be changed by the   *   FILE 441
//*            RACF Security Administrator as soon as practicable   *   FILE 441
//*            after use.                                           *   FILE 441
//*                                                                 *   FILE 441
//*     RA#NAMES                                                    *   FILE 441
//*     ========                                                    *   FILE 441
//*       DATE - October 1990.                                      *   FILE 441
//*              Jan 1991 - Automatic REVOKE feature added.         *   FILE 441
//*              Feb 2000 - Fixed minor Y2K bug in report header    *   FILE 441
//*                         and major Y2K bug in automatic REVOKE   *   FILE 441
//*                         feature.  Added "revoke trace"          *   FILE 441
//*                         sub-feature.                            *   FILE 441
//*       FUNCTIONS -                                               *   FILE 441
//*       1. Write a list of all userids and their respective NAME  *   FILE 441
//*          fields.  The output DDname is UIDS.                    *   FILE 441
//*       2. In the case of userids which have not been used yet,   *   FILE 441
//*          if a number of days have elapsed since creation, set   *   FILE 441
//*          the revoke indicator (FLAG4).  This "number of days"   *   FILE 441
//*          is specified as a the parameter when invoking this     *   FILE 441
//*          program.                                               *   FILE 441
//*       3. Write a list of all group names and their respective   *   FILE 441
//*          installation-defined data fields.                      *   FILE 441
//*          The output DDname is GRPS.                             *   FILE 441
//*                                                                 *   FILE 441
//*       USER DOCUMENTATION -                                      *   FILE 441
//*          //RA#NAMES EXEC PGM=RA#NAMES,PARM='31'                 *   FILE 441
//*          //UIDS     DD   DSN=SYS3.RACFADM.USERIDS,DISP=SHR      *   FILE 441
//*          //GRPS     DD   DSN=SYS3.RACFADM.GROUPS,DISP=SHR       *   FILE 441
//*          (Supply any sequential data sets; this program has     *   FILE 441
//*           the DCB attributes hard-coded).                       *   FILE 441
//*       1. The PARM value is the number of days before an unused  *   FILE 441
//*          (new) userid will be automatically REVOKEd by this     *   FILE 441
//*          program.  Maximum value is 3 decimal digits.           *   FILE 441
//*       2. If there is no parameter, or a value of zero is        *   FILE 441
//*          specified, no REVOKE is performed.                     *   FILE 441
//*       3. If the value is preceded by a minus sign, e.g.         *   FILE 441
//*          PARM='-31' the REVOKE is not actually performed, but   *   FILE 441
//*          trace WTOs indicate what would have happened if the    *   FILE 441
//*          minus sign had been omitted.                           *   FILE 441
//*                                                                 *   FILE 441
```
