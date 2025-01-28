#### [Занятое место на диске топ 20](https://www.virtono.com/community/tutorial-how-to/how-to-find-the-largest-files-in-linux/)
```bash
sudo du -ah / | sort -rh | head -n 20
find $HOME -type f -printf '%s %p\n' | sort -nr | head -10
```
