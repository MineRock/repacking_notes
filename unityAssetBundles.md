# Intro
Hi. I'm MineRocker, a former repacker. I am writing this to log everything that I learned on my journey. This one is a special one because it took me a long time to figure out.

Disclaimer - All files on which the following operations were performed were obtained legitimately and I do not condone getting them from illegal sources (wink wink).

## Purpose
The purpose of the current document is to decompress and then recompress (with a different compression algorithm) the unity AssetBundles. Why are we doing this? Let me take the example of Surgeon Simulator 2, a game I repacked in the past. This game ships with 3000 files in the AssetBundles folder that sum up to about 1.66GiB. We can decompress these files and then recompress them using AssetsTools.NET.

In the past, I decompressed the files in advance using [UABE](https://github.com/SeriousCache/UABE), repacked the game using the decompressed files, and then added a script to re-compress the files with LZMA and rename them properly post-install. This method is pretty inefficient, and since re-compressing them in advance would get you the same file size in the end (and perhaps a shorter file size for the repack), I will write a script that decompresses and then recompresses that file in advance using [AssetsTools.NET](https://github.com/nesrak1/AssetsTools.NET).

### Steps
* Install Visual Studio and install the .NET desktop collection on the installer page. It is about 7.5GB in size.
* Create a new project - C# console app.
* After the project loads, to your right you will see a Solution Explorer that has a 'Dependencies' entry. Right click, Manage NuGet Packages, search for 'AssetsTools.NET' by nesrak1 and install it. Right now, the latest stable is 3.0.0.
* The following is a demonstration of how you can use the above library:
```csharp
using AssetsTools.NET;
using AssetsTools.NET.Extra;

String filename = "04283348d8fcb7536b5d72c8cea763a6"; // It is a 173MB file that I will use for demonstration purposes.

var am = new AssetsManager();
var og = am.LoadBundleFile("C:\\Seedbox\\Surgeon Simulator 2\\Surgeon Simulator 2\\Surgeon Simulator 2_Data\\StreamingAssets\\Content\\AssetBundles\\" + filename);

using (var st = File.OpenWrite("C:\\Users\\MineRocker\\decomp\\" + filename))
using (var writer = new AssetsFileWriter(st))
{
    og.file.Unpack(writer);
} // The file reading thread gets closed here so that the file is not locked anymore

var unpacked = am.LoadBundleFile("C:\\Users\\MineRocker\\decomp\\" + filename);

using (var strcmp = File.OpenWrite("C:\\Users\\MineRocker\\decomp\\" + filename + "-recomp"))
using (var writerrcmp = new AssetsFileWriter(strcmp))
{
    unpacked.file.Pack(writerrcmp, AssetBundleCompressionType.LZMA);
} // The file reading thread gets closed here so that the file is not locked anymore
```

The file goes from 173MB to 124MB. This code does not check whether the folder in which we are decompressing the AssetBundle exists, you can add that in later if you would like to do so.

### Extra Notes
The original files (from the developers) were compressed using LZ4. We compressed them with LZMA.

Analyzing the decompressed file with GameFileScanner (scanning for ZLib with Dynamic Streams), I can see that there's about 2 million ZLib streams that could get the size from 16.7MB to 48.6MB. This might look against our goals of compression but this will in turn help it compress better. On compressing the file with xtool(zlib+reflate)+srep+lolz, the file goes to 56.2MB and takes 7 seconds to unpack. The file hashes are exactly the same.

If you try to compress the "recompressed" file, let us first analyze the file. It still has about 400000 ZLib streams that would get the size from 3.22MB to 6.79MB. Compression could prove to be helpful but let us see. Unfortunately as I expected, lolz is unable to get any more compression out of this (to be fair it is compressed with LZMA already) and the final file size comes out to be slightly bigger than the original file (lmao) - 124MB + spare change.

## Conclusion
It is fair to say that this method brings great benefits to compression, but for the ideal case, one would have to add the decompressor and execute it after installation, the added size could be pretty significant for smaller games (packaging the runtime into the executable brings it to about 60MB) but for larger games you could add it to your repack and just delete it later. The difference you get from adding the decompressed files to the repack far outweighs the 60MB. Anyways, that's all I had. This was a fun ride, thank you for following this guide.
