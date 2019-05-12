# Discussing the `SettingwithCopyWarning` in Pandas

Link to presentation: https://gitpitch.com/tvasil/settingwithcopywarning#/

Presentation made for "SettingwithCopyWarning" in Pandas. 

## Resources:
- [Pandas documentation - indexing](https://pandas-docs.github.io/pandas-docs-travis/user_guide/indexing.html)
- [More specifically on copy vs view](https://pandas-docs.github.io/pandas-docs-travis/user_guide/indexing.html#returning-a-view-versus-a-copy)
- [SettingwithCopyWarning: How to Fix This Warning in Pandas](https://www.dataquest.io/blog/settingwithcopywarning/) - DataQuest
- [Syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) - Wikipedia
- [Views, Copies, and the SettingWithCopyWarning Issue](https://github.com/pandas-dev/pandas/issues/10954) - Issue on GitHub where the future of Pandas is highly questioned in relation to this Warning. 

"[There are no cases](https://github.com/pandas-dev/pandas/issues/8730#issuecomment-61717234) that I am aware that you should actually ignore this warning. Your code may still work, that's why its a warning. It has to do with whether what you are modifying is actually a copy of a view. In general if its a single dtyped frame it mostly will work, BUT NOT ALWAYS. And that's the rub. If you do certain types of indexing it will never work, others it will work. You are really playing with fire." - [Jeff Reback](https://github.com/jreback), one of the main contributors to Pandas
