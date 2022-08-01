#!/bin/bash
upload(){
    hugo --theme=even --baseUrl="https://hzjsea.github.io/"
    cd public ||exit 1
    git init
    git remote add origin https://github.com/hzjsea/hzjsea.github.io.git 
    git add -A
    git commit -m "first commit"
    git push -u origin master -f
}

# example: hugo new post/first.md
newFile(){
    hugo new "$1"
}


if [ "$1" ]; then
   if [[ "$1" == "upload" ]]; then
        upload
   fi
   
   if [[ "$1" == "new" ]];then
        newFile "$2"
   fi
fi
