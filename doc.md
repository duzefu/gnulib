RICHACLEX(7)		   Rich Access Control Lists		  RICHACLEX(7)



NAME
       richaclex - RichACL Examples

DESCRIPTION
       This  man-page demonstrates the various features of Rich Access Control
       Lists (RichACLs) by example, and shows how they interact with the POSIX
       file permission bits.

       For  a  complete description of the structure, concepts, and algorithms
       involved, please refer to richacl(7).

EXAMPLES
   Traditional POSIX file permissions as RichACLs
       In the traditional POSIX file permission model, each file and directory
       has  a file mode that specifies the file type and file permission bits.
       The file permission bits	 determine  the	 permissions  for  the	owner,
       group,  and  other classes of processes.	 The owner class includes pro-
       cesses which are the file owner. The  group  class  includes  processes
       which  are  not the file owner and which are either a user mentioned in
       an ACL, or which are in the owning group or in a group mentioned in  an
       ACL.  The other class includes all processes which are not in the owner
       or group class.

       In the absence of inheritable permissions, when a file or directory  is
       created,	 the  effective	 file permissions of the new file or directory
       are (mode & ~umask), where mode	is  the	 mode  parameter  to  open(2),
       mkdir(2), and similar, and umask is the process umask (see umask(2)):
```
	   $ umask
	   0022
	   $ touch f
	   $ ls -l f
	   -rw-r--r-- 1 agruenba users 0 Feb 24 09:37 f
```


       Here,  the umask has a value of 022. The touch(1) command calls open(2)
       with a mode parameter of 0666 to create a new file, and	the  resulting
       effective  file	permissions are 0644, displayed as rw-r--r-- by ls(1):
       the owner has read and write access, and the group  and	other  classes
       have read access only.

       These permissions are displayed as a RichACL as follows:
```
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow
	    everyone@:r------------::allow

	   $ getrichacl --long f
	   f:
	       owner@:read_data/write_data/append_data::allow
	    everyone@:read_data::allow

```


       RichACLs have both a short text form in which each permission is repre-
       sented by a single letter, and a long text form in which	 each  permis-
       sion is represented by an equivalent string.  The owner@ special inden-
       tifier refers to the file owner, an the	everyone@  special  identifier
       refers to everyone including the owner and the owning group.  The POSIX
       read permission maps to the RichACL read_data (r) permission; the POSIX
       write permission maps to the RichACL write_data (w) and append_data (p)
       permissions.

       Creating directories works similarly:
```
	   $ mkdir d
	   $ ls -dl d
	   drwxr-xr-x 2 agruenba users 4096 Feb 24 09:37 d

```

       Here, the mkdir(1) command calls mkdir(2) with a mode parameter of 0777
       to create a new directory, and the resulting effective file permissions
       are 0755, displayed as rwxr-xr-x by ls(1).  The owner has  read,	 write
       and  execute (search) access, and the group and other classes have read
       and execute (search) access.

       These permissions are displayed as a RichACL as follows:
```
	   $ getrichacl d
	   d:
	       owner@:rwpxd--------::allow
	    everyone@:r--x---------::allow

	   $ getrichacl --long d
	   d:
	       owner@:list_directory/add_file/add_subdirectory/execute/delete_child::allow
	    everyone@:list_directory/execute::allow

```


       For  directories,  the  POSIX  read  permission	maps  to  the  RichACL
       list_directory (r)  permission;	the POSIX write permission maps to the
       RichACL add_file (w), add_subdirectory (p), and	delete_child (d)  per-
       missions;  the  POSIX  execute  (search) permission maps to the RichACL
       execute (x) permission. The long text forms for some of the permissions
       differ  between	files  and directories. When setting ACLs, either form
       can be used interchangeably.

       When the file permission bits of a file are set to the unusual value of
       0604,  the  owning  group  will not have read access, but everyone else
       will.  This maps to the following RichACL:
```
	   $ chmod 604 f
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow
	       group@:r------------::deny
	    everyone@:r------------::allow

```


       A deny entry for the owning group (group@) before the final allow entry
       for   everyone	else   indicates  that	the  owning  group  is	denied
       read_data (r) access.

   RichACLs
       RichACLs can be used for granting users and groups  additional  permis-
       sions,  or for denying them some permissions. This includes permissions
       that go beyond what can be  granted  by	the  traditional  POSIX	 read,
       write,  and execute permissions.	 The following example grants user Tim
       the right to read, write, and append to a file, to  change  the	file's
       permissions   (write_acl (C)),  and  to	take  ownership	 of  the  file
       (write_owner (o)):
```
	   $ touch f
	   $ ls -l f
	   -rw-r--r-- 1 agruenba users 0 Feb 24 09:37 f
	   $ setrichacl --modify user:tim:rwpCo::allow f
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow
	    everyone@:r------------::allow
	     user:tim:rwp------Co--::allow

	   $ ls -l f
	   -rw-rw-r--+ 1 agruenba users 0 Feb 24 09:37 f
```


       Setting the ACL has updated the file permission bits, and ls(1) shows a
       +  sign after the file permissions to indicate that the file now has an
       ACL. The change in file permission bits indicates that one or more mem-
       bers  of	 the  group  class now have POSIX write access, or a subset of
       POSIX write access  (in	this  case,  the  RichACL  write_data (w)  and
       append_data (p) permissions).

       The  group class permissions are not the same as the permissions of the
       owning group; the owning group still only has read_data (r) access.

       In general, when the ACL of a file or directory is  changed,  the  file
       permission  bits are updated to reflect the maximum permissions of each
       of the file classes as closely as possible.  Permissions that go beyond
       the POSIX read, write, and execute permissions are not reflected in the
       file permission bits.

   Changing the file permission bits
       When the file permission bits of a file or directory are	 changed  with
       chmod(2),  POSIX	 requires that the new file permission bits define the
       maximum permissions that any process is granted.	 Therefore,  when  the
       file permission bits of file f from the previous example are changed to
       0664 (their current value), the following happens:
```
	   $ chmod 664 f
	   $ ls -l f
	   -rw-rw-r--+ 1 agruenba users 0 Feb 24 09:37 f
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow
	     user:tim:rwp----------::allow
	    everyone@:r------------::allow

```


       User Tim loses the write_acl (C) and  write_owner (o)  permissions.  In
       addition,  the  entry for the special identifier everyone@ moves to the
       end of the ACL; this does not  change  the  permissions	that  the  ACL
       grants.

       When  the  file permission bits are changed so that only the file owner
       has access to the file, the ACL changes in the following way:
```
	   $ chmod 600 f
	   $ ls -l f
	   -rw-------+ 1 agruenba users 0 Feb 24 09:37 f
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow

```


       The ACL reflects that user Tim and the special identfier	 everyone@  no
       longer  have  access  to the file. The permissions prevously granted by
       the ACL have not entirely disappeared, they are merely  masked  by  the
       new  file  permission  bits,  though  (by  way  of  the file masks; see
       richacl(7)).  When the file permission bits are changed back  to	 their
       previous value, those permissions become effective again:
```
	   $ chmod 664 f
	   $ ls -l f
	   -rw-rw-r--+ 1 agruenba users 0 Feb 24 09:37 f
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow
	     user:tim:rwp----------::allow
	    everyone@:r------------::allow
```



       When  the file permission bits are changed to the value 0666, we end up
       with the following result:
```
	   $ chmod 666 f
	   $ ls -l f
	   -rw-rw-rw-+ 1 agruenba users 0 Feb 24 09:37 f
	   $ getrichacl f
	   f:
	       owner@:rwp----------::allow
	     user:tim:rwp----------::allow
	       group@:-wp----------::deny
	    everyone@:rwp----------::allow

```


       By giving POSIX write access to the other class, the everyone@  special
       identifier  has gained write_data (w) and append_data (p) access to the
       file. The owning group still only has read_data (r) access, though.

   Inheritance of permissions at file-creation time
       When a file or directory is created, the ACL of	the  parent  directory
       defines	which  of  the	parent's ACL entries the new file or directory
       will inherit: files will inherit entries with the file_inherit (f) flag
       set.  Directories  will	inherit	 entries with the dir_inherit (d) flag
       set.  In	 addition,  directories	 will	inherit	  entries   with   the
       file_inherit (f)	 flag  set  as	inheritable-only permissions for their
       children; the inherit_only (i) flag is set. The	no_propagate (n)  flag
       can be used to inherit permissions one level deep only.

       When a file or directory inherits permissions, the file permissions are
       determined by the mode parameter as given  to  open(2),	mkdir(2),  and
       similar,	 and  by  the  inherited  permissions;	the process umask (see
       umask(2)) is ignored.

       The following example creates a directory and sets up inheritable  per-
       missions	 for  files  and  subdirectories  (the example is indented and
       padded with dashes for improved readability):
```
	   $ mkdir d
	   $ setrichacl --set '
	   >	  owner@:rwpxd:fd:allow
	   >	user:tim:rwpxd:fd:allow
	   > group:staff:r--x-:f-:allow
	   >   everyone@:r--x-:fd:allow' d
```


       Of the four ACL entries, three are inheritable for files	 and  directo-
       ries  (the file_inherit (f) and dir_inherit (d) flags are set), and the
       entry  for  group  Staff	  is   inheritable   for   files   only	  (the
       dir_inherit (d) flag is not set).

       Files created inside d inherit the following permissions:
```
	   $ touch d/f
	   $ ls -l d/f
	   -rw-rw-r--+ 1 agruenba users 0 Feb 24 09:37 d/f
	   $ getrichacl d/f
	   d/f:
		 owner@:rwp----------::allow
	       user:tim:rwp----------::allow
	    group:staff:r------------::allow
	      everyone@:r------------::allow

```


       The  touch(1)  command  calls  open(2) with a mode parameter of 0666 to
       create a new file, so the execute (x) permission is masked by the  file
       permission bits. When the file permission bits are changed to the value
       0775 with chmod(1), the ACL changes as follows:
```
	   $ chmod 775 d/f
	   $ getrichacl d/f
	   d/f:
		 owner@:rwpx---------::allow
	       user:tim:rwpx---------::allow
	    group:staff:r--x---------::allow
	      everyone@:r--x---------::allow

```


       Directories created inside d inherit the following permissions:
```
	   $ mkdir d/d
	   $ ls -dl d/d
	   drwxrwxr-x+ 2 agruenba users 4096 Feb 24 09:37 d/d
	   $ getrichacl d/d
	   d/d:
		 owner@:rwpxd--------:fd:allow
	       user:tim:rwpxd--------:fd:allow
	    group:staff:r--x---------:fi:allow
	      everyone@:r--x---------:fd:allow

```


       The inherit_only (i) flag of the entry for group Staff is set to	 indi-
       cate  that the entry has no effect on the effective permissions of d/d.
       When a file is created inside d/d, the  inherit_only (i)	 flag  in  the
       entry  inherited by the file is cleared again, and the resulting ACL is
       somilar to that of d/f.

   Inheritance of file permission bits only
       When the permissions inherited by  a  new  file	or  directory  can  be
       exactly	represented by the file permission bits, only the file permis-
       sion bits of the new file or directory will be set, and no ACL will  be
       stored (no + sign is shown after the file permission bits):
```
	   $ ls -dl d
	   drw-------+ 3 agruenba users 4096 Feb 24 09:37 d
	   $ getrichacl d
	   d:
	    owner@:rwp----------:f:allow

	   $ touch d/f
	   $ ls -l d/f
	   -rw------- 1 agruenba users 0 Feb 24 09:37 d/f
	   $ getrichacl d/f
	   d/f:
	    owner@:rwp----------::allow
```



   Automatic Inheritance
       The NFSv4 and SMB network protocols support creating files and directo-
       ries without specifying any permissions for the new file or  directory.
       When   the   directory  in  which  such	a  file	 is  created  has  the
       auto_inherit (a) ACL flag set, then the new files and directories  cre-
       ated  in the directory will have the auto_inherit (a) ACL flag set, and
       each ACL entry inherited from the parent directory will have the inher-
       ited (a) flag set. For example, consider the following directory:
```
	   $ ls -dl d
	   drw-rw----+ 2 agruenba users 4096 Feb 24 09:37 d
	   $ getrichacl d
	   d:
	       flags:a
	      owner@:rwp----------:f:allow
	    user:tim:rwp----------:f:allow

```


       When  a	file  is  created inside that directory without specifying any
       file permissions, the file inherits the following ACL:
```
	   $ getrichacl d/f
	   d/f:
	       flags:a
	      owner@:rwp----------:a:allow
	    user:tim:rwp----------:a:allow

```


       When the ACL of the directory is then changed, those changes  propagate
       to the file:
```
	   $ setrichacl --modify group:staff:r:f:allow d
	   $ getrichacl d/f
	   d:
		  flags:a
		 owner@:rwp----------:a:allow
	       user:tim:rwp----------:a:allow
	    group:staff:r------------:a:allow

```


       When ACL entries are propagated from a directory to one of its children
       (the files and directories inside the directory), all  entries  in  the
       child's	ACL  that have the inherited (a) flag set are removed, and the
       child inherits ACL entries from the parent in the same way  as  when  a
       new  file  or  directory	 is  created.  The inherited (a) flag of those
       inherited entries is set. Existing entries in the child's ACL  that  do
       not have the inherited (a) flag set are left untouched:
```
	   $ setrichacl --modify user:ada:rwp::allow d/f
	   $ getrichacl d/f
	   d:
		  flags:a
	       user:ada:rwp----------::allow
		 owner@:rwp----------:a:allow
	       user:tim:rwp----------:a:allow
	    group:staff:r------------:a:allow

	   $ setrichacl --modify group:staff:::allow d
	   $ getrichacl d
	   d:
	       flags:a
	      owner@:rwp----------:f:allow
	    user:tim:rwp----------:f:allow

	   $ getrichacl d/f
	   d:
	       flags:a
	    user:ada:rwp----------::allow
	      owner@:rwp----------:a:allow
	    user:tim:rwp----------:a:allow

```


       We  remove the allow entry for group Staff from the ACL by assigning it
       an empty set of permissions.

       When the file permission bits of a file or directory are	 changed  with
       chmod(2),  the  Automatic Inheritance algorithm must no longer override
       those permissions.  Likewise, when a file or directory is created  with
       open(2), mkdir(2), or similar, the mode parameter to those system calls
       defines an upper limit to the file permission bits of the new  file  or
       directory, and the Automatic Inheritance algorithm must no longer over-
       ride the resulting permissions.	To achieve that, when the ACL  of  the
       file  or	 directory has the auto_inherit (a) flag set, those operations
       set the protected (p) flag, which stops the Automatic Inheritance algo-
       rithm from modifying the ACL:
```
	   $ chmod 660 d/f
	   $ getrichacl d/f
	   d/f:
	       flags:ap
	    user:ada:rwp----------::allow
	      owner@:rwp----------:a:allow
	    user:tim:rwp----------:a:allow

	   $ touch d/g
	   $ getrichacl d/g
	   d/g:
	       flags:ap
	      owner@:rwp----------:a:allow
	    user:tim:rwp----------:a:allow

```


   Effective permissions
       With complex ACLs, it can become difficult to determine the permissions
       of a particular user or group. In this situation, the  --access	option
       of getrichacl(1) can be used:
```
	   $ getrichacl --access d/f
	   rwpx---------  d/f
	   $ getrichacl --access=tim d/f
	   rwpx---------  d/f
	   $ getrichacl --access=:staff d/f
	   r--x---------  d/f

```

       When the --access option is used without arguments, getrichacl displays
       the permissions the current process  has	 for  the  specified  file  or
       files.  With  a user name as the argument, getrichacl displays the per-
       missions of that user. With a colon followed  by	 a  group  name,  get-
       richacl displays the permissions of that group.

AUTHOR
       Written by Andreas Gruenbacher <agruenba@redhat.com>.

       Please  send  your  bug reports, suggested features and comments to the
       above address.

CONFORMING TO
       Rich Access Control Lists are Linux-specific.

SEE ALSO
       getrichacl(1), setrichacl(1), richacl(7)
