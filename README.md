# CachéRDD

The real-time RDBMS RDD for the Harbour family of compilers.

CachéRDD is a RDD (Replaceable Database Driver) for the open-source [Harbour](http://www.harbour-project.org) and [xHarbour](http://www.xharbour.org) compilers to connect to [InterSystems](http://www.intersystems.com) Caché, the post-relational, massively scalable database.

CachéRDD is a classic blend of true RDBMS and time-tested traditional xbase syntax:

- the same SEEK, the same SKIP, the same GOTO, GOTOP, GOBOTTOM, the same INDEX ON, the same TBROWSE - and that too with billions of records, with the same SPEED, even faster in some cases...
- the same RLOCK, the same FLOCK, even extended to be honored by external processes, the same USE EXCLUSIVE / SHARED

The only difference is that now your data is more SECURE, SCALABLE, FREE OF INDEX CORRUPTION, with TRANSACTION CONTROL, and subject to INTEGRATION WITH A WIDE RANGE OF EXTERNAL SQL TOOLS.

And at top of all you need not to change a single line of source except for resolving table name issues and adjusting a few index expressions here and there.

CachéRDD does not issue any SQL commands to the database for any of the RDD functions, nor does it need a DSN (Data Source Name). CachéRDD does not require ODBC drivers to be installed. CachéRDD talks to the database directly with its lower-level interface. This is the main reason that DBFCDX-like functionality has been achieved.

## In production since 2007

From the author's notes in `ChangeLog.txt` (initial commit, August 2015):

- CachéRDD has been in production at CURACAO since 2007. In peak season 950+ concurrent instances of the application connect to Caché without any issues.
- At the time of publishing: some 325+ tables on Caché, a 222+ GB database serving 525+ million records.
- Since 2007 only one table had to be reindexed - because of a flaw in the application, not an RDD or Caché issue.

## Benefits

As listed by the author on the documentation site:

1. **Scalability** - a very high rate of data-sharing with other agencies, beyond what proprietary xBase table structures allow.
2. **Platform independence** - `CACHE.DAT`, be it CachéRDD compiled code or application data, is portable across platforms.
3. **No more index corruption** - indexes live in the database, not in files that corrupt in networking environments.
4. **Transaction control** - either a transaction is posted in its entirety or nothing is changed in the database.
5. **No limitation on table size or number of records** - beyond the 2 GB / ~400 million records wall of native (x)Harbour RDDs.
6. **Adaptation to newer technologies** - keep the xbase syntax for base development and adopt newer arrivals to connect to the same database.

## Repository layout

| Path | What it is |
| ---- | ---------- |
| `cacherdd.prg`, `cachefunc.prg`, `cacheqry.prg`, `cachedic.prg`, `cacheutils.prg` | The RDD sources plus extended functions important and/or useful for its implementation |
| `cacherdd.ch`, `cacherdd.hbc`, `cacherdd.hbp`, `cacherdd.hbx` | Header and hbmk2 project files |
| `cache.c`, `c_api.h` | C binding sources; `c_api.h` is InterSystems property (see `COPYING.txt`) |
| `bin/` | Communication libraries `cbind.dll`, `CacheODBC.dll`, `cppbind_msvc80.dll` from Caché 2007.1.0.369 - InterSystems property, included because 2007.1 is discontinued and higher versions never worked with CachéRDD; good for all versions of Caché from 2007 until now |
| `server/` | Server-side sources (`Rdd.mac`, `Vouch.inc`, `Vouch.XhbRdd.cls`) and the compiled object code `server/rdd/2015_1/CACHE.DAT` (compiled with Caché 2015.1, cross-platform) |
| `cachemgr/` | CacheMGR, a DBU-like application with many more goodies to manage every aspect of your tables without visiting the server |
| `readme.txt`, `COPYING.txt`, `ChangeLog.txt` | Original notes, InterSystems binaries disclaimer, and history |

## Building and deployment

- Harbour contrib layout, built with hbmk2 (`cacherdd.hbc` / `cacherdd.hbp`); tested with BCC and MinGW on Windows. The binaries in `bin/` are for Windows clients.
- Server side: create the CACHERDD database and namespace, copy the supplied `CACHE.DAT` (or compile from the sources in `server/` and transfer it), and map the RDD package and routines to your data namespaces. Full steps are on the documentation site.

## Behavior notes

From the documentation site:

- **Locks.** `RLOCK()` and `FLOCK()` are honored by external processes as well. Once `RLock()` is successful, no other process, RDD-oriented or not, can update that record.
- **Transactions.** Supported via `CacheBeginTransaction()` / `CacheEndTransaction()` / `CacheRollBackTransaction()`. Nested transactions only partially - avoid them until the documentation says otherwise.
- **Record numbers** are assigned by Caché when a record is inserted and never change for the lifetime of the table. `LastRec()` returns the number of records inserted and never takes deleted records into account.
- **Phantom record** is reached with `GO BOTTOM` + `SKIP` (or `CacheGotoPhantom()`), not `GO LastRec()+1`.
- **Deleted records** are gone permanently, by design, so RDD applications can co-exist with external non-RDD tools over ODBC/JDBC/.NET. Management of deleted records is possible another way, documented on the site.
- **Schemas** arrange tables like folders (default schema `SQLUSER`); the `VOUCH||MYTABLE.DBF` syntax picks a schema at `DbCreate()` / `USE` time.
- **RDD function coverage:** most of the documented RDD functions are implemented; a few are partial (`DbInfo()`, `DbOrdInfo()`) or not implemented (relations, `DbRecall()`/`Deleted()` as such, condition/order-bag functions). The full status table is on the site.

## Documentation

- Portal: <https://harbour-addons.github.io/cacherdd/>
- Original documentation site: <http://cacherdd.sourceforge.net>
- Original home (SourceForge, SVN): <https://sourceforge.net/projects/cacherdd/>

## License

GNU General Public License v2 or later, with the Harbour linking exception (see the header of `cacherdd.prg`).

The InterSystems binaries and `c_api.h` are the property of InterSystems and are included for application development only. See `COPYING.txt`.

## Credits

- Written by **Pritpal Bedi** (<bedipritpal@hotmail.com>). Initiated in 2006 with xHarbour; some sources respect xHarbour symbols, hashes and try/catch, kept as is.
- Published with the kind permission of **CURACAO** (<http://www.icuracao.com>), with thanks to **Eric Baruch** (the then CFO) and **Eryk Szachniewicz** (Manager, MIS).
- Built on top of **UsrRDD** - thanks Przemek for this superb tool.
