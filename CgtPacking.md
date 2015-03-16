# The CGT size problem #

The CGT file format as defined by Devin Cook is well structured. Each grammar object has its own record data. The charset record is basically a string of all characters which are to be used in the charset.

When Unicode text is to be parsed, those charsets can become very large, even though they often cover just a range of many characters. Therefore, the CGT file grows quickly if such character sets are being used (especially with `{All Valid}` charsets).

# Proprietary solution for this engine #

This engine (version V1.2 or newer) can deal with a new record type: the packed charset. Basically, the record has two additional integer entries which define start and end of the largest character range used for this charset. The characters not in this range are handled just like before as string of characters.