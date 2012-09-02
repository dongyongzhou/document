---
layout: master
title: ruby-file-operation
---

## Overview


## find file 

    def findfiles(dir,name)
    list=[]
    Find.find(dir) do |path|
        puts path
        Find.prune if [".",".."].include?path
       logger.info path
            case name
                when String
                        list << path if File.basename(path) == name
                when Regexp
                        list << path if File.basename(path) =~name
        else
                raise ArgumentError
        end
        end
        list
    end

## zip dir

