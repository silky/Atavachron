# ✇ Atavachron

## Fast, scalable and secure de-duplicating backup

- Scalable to any repository size (stream-based architecture)
- Verifiable immutable snapshots that are forever incremental
- Content-derived chunking for optimal de-duplication
- Encryption (libsodium) and compression (LZ4)
- Lock-free repository sharing and de-duplication across multiple machines
- Multi-threaded chunk processing and upload
- Property-based testing of core processing pipeline
- Amazon S3 support

*WARNING: Currently under active development and not yet ready for use*

## FAQ

### Why not just use DropBox?

The important reasons for me, which apply to most consumer cloud storage offerings, are the cost of these services relative to Amazon S3, file count limits and the difficulty of integrating trustworthy client-side encryption. I also do not want bi-directional sync, which is a complex problem and difficult to get right. DropBox does offer online fine-grained incremental backup, sending changes to the cloud one file at a time. They also use de-duplication, but it is for their benefit only.
Atavachron is a different point in the design space. In order to handle potentially huge amounts of files and maximise upload performance, it packs them into chunks. This also has the advantage of hiding the file sizes from the remote repository, which makes the encryption more secure. Atavachron backups are forever incremental, but there is potentially more cost in terms of storage space for each backup performed (depending on the chunk size chosen).

### Why not use an existing backup program for Amazon S3?

They may be more appropriate in many cases. However, typically other backup programs are not *scalable*, which can be a problem for large amounts of data and machines with limited memory.

### What exactly do you mean by scalable?

Atavachron is scalable because the problem size (i.e. the number of files to be backed up) is not limited by the available memory of the machine performing the backup. It should be possible to backup terabytes of data using only a few hundred megabytes of working memory. That said, there are some minor limitations, for example we do need to realise all the file names of a particular directory in memory in order to sort them and diff them. However, this is unlikely to be an issue in practice.

### What exactly do you mean by fast?

I believe it should be fast enough. The primary goals are clarity and correctness of the implementation. We will almost certainly take a performance hit by being scalable, as we do not use in-memory data structures for holding chunk sets and file lists.
Atavachron can chunk, hash, compress and encrypt the entire Linux source code repository, as of May 2018, to an alternative local folder in about 30 seconds on my old Thinkpad. Subsequent backups containing a few changes take seconds. When backing up to a remote repository such as Amazon S3, I suspect the throughput will be limited more by the performance of the network and the remote server.

### How does de-duplication work?

Atavachron uses content-derived chunking (CDC) for de-duplication. A rolling hash with per-repository secret parameters is used to derive boundaries in the packed file data. The idea is that for many types of file change, for example inserting bytes at the beginning, it should not be necessary to re-write many chunks. The parameters are chosen to give the desired statistical distribution of chunk size.
The chunks are then hashed using a secret key; and then, only if necessary, compressed, encrypted and uploaded. The chunks are stored in the remote repository, using their hashes as file names. Any new chunk with a matching hash can be identified as a duplicate. This works for the entire repository across multiple machines and backups.

### How secure is it?

The highly regarded *Libsodium* provides the high-level APIs for use by cryptography non-experts such as myself. Atavachron hashes using HMAC-SHA512 and encrypts using an XSalsa20 stream cipher with Poly1305 MAC authentication.

### Why Haskell?

Haskell offers a level of type-safety and expressiveness that is unmatched by most other practical languages. GHC Haskell is also capable of producing highly performant executables from very high-level abstract code. Atavachron has been written largely by composing transformations on effectful on-demand streams, resulting in better modularity and separation-of-concerns when compared to more traditional approaches. The high-level pipeline architecture should be visible in the source file "src/Atavachron/Pipelines.hs".

### What is property-based testing?

Property-based tests consist of assertions and logical properties that a function should fulfil. These properties are tested for many different randomly-generated function inputs, essentially generating test cases automatically. The canonical property-based testing framework is Haskell's QuickCheck.

### Will there be native Windows support?

This isn't currently planned. However, I do intend to test it under the Windows 10 Subsystem for Linux (WSL).

### Where does the name come from?

The Atavachron was the time machine that Kirk, Spock and McCoy unwittingly used in the Star Trek episode "All our Yesterdays". Atavachron is also a really good 80's jazz fusion album by the late great guitarist Allan Holdsworth.