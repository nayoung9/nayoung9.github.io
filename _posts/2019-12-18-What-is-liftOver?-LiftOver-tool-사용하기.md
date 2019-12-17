---
layout: post
title: What is liftOver? / Liftover tools 사용하기!
---
## Liftover 란?

같은 종의 서로 다른 버전의 유전체 정보를 활용해서 내가 가지고 있는 정보의 다른 버전상의 정보로 변환하는 작업(?)이다. 사람 유전체를 수능특강 교재에 비유하면(..) 수능특강 교재는 매 해 새로운 정보들이 추가되거나, 기존 정보중에 일부가 빠지면서 교체되어 다음 해 수능특강 교재로 버전업이 되는데, 1단원을 집합으로 시작해서 연이어 나오는 내용의 뼈대는 크게 같다. 인강 강사들이 수능특강을 가지고 자기만의 강의자료를 만든다고 할 때, 이 부분은 수능특강 a~b p  참고와 같은 주석을 붙이는데, 학생 입장에서 작년에 결제하고 얻은 교재를 재수할 때 또 써먹으려고 새 교재에서 a~b 페이지를 찾고 싶은 상황에!  liftOver tool을 활용하면 이전 교재의 a~bp → 현재 교재의 c~dp 를 얻을 수 있겠다! 

**작년 인강강사의 교재 속 주석 데이터를 → 현재 수특 교재에 맞도록 변환하는 작업**

이 유전체에 대해 이루어진다고 생각할 수 있겠다.

**hg19 를 기준으로 만들어진 유전자 위치 데이터를 → hg38을 기준으로 그 위치를 변환하는 작업(*hg19, hg38 모두 인간 유전체의 서로 다른 버전이다.)**

이 작업을 하기 위해서는 **서로 다른 버전의 데이터가 어떻게 연관되어있는지** 알려줄 수 있는 데이터가 추가적으로 필요하겠다. 수능특강의 예를 마저 들면 두 책의 목차를 비교해서

2019 교재의 1단원 a page —- 2020 교재의 1단원 b page 

2019 교재의 2단원 c page —- 2020 교재의 2단원 d page 

과 같은 정보를 함께 주면 변환할 수 있을 것이고, 유전체의 경우 다른 두 버전의 유전체 데이터의 **Alignment** 정보를 주면 변환할 수 있다. (Alignment데이터를 통해 서로 다른 두 버전의 유전체를 동일한 부분이 같은 column 에 오도록 배열해서 서로 다른 버전의 유전체 각각의 지역이 매치된다는 정보를 넘겨줄 수 있다.

UCSC genome browser 에서 제공하는 liftOver tool 이 있고, 

[Lift Genome Annotations](https://genome.ucsc.edu/cgi-bin/hgLiftOver)

이와 동일한 기능을 수행하도록 개발된 파이썬 모듈인 CrossMap 이 있다.

[What is CrossMap ? - CrossMap 0.4.0 documentation](http://crossmap.sourceforge.net/)

---
## Liftover tool 사용하기!
연구실에서는 주로 liftOver tool을 활용하므로 이를 좀 더 자세히 살펴보면 

    liftOver - Move annotations from one assembly to another
    usage:
       liftOver oldFile map.chain newFile unMapped
    oldFile and newFile are in bed format by default, but can be in GFF and
    maybe eventually others with the appropriate flags below.
    The map.chain file has the old genome as the target and the new genome
    as the query.
    
    ***********************************************************************
    WARNING: liftOver was only designed to work between different
             assemblies of the same organism. It may not do what you want
             if you are lifting between different organisms. If there has
             been a rearrangement in one of the species, the size of the
             region being mapped may change dramatically after mapping.
    ***********************************************************************
    
    options:
       -minMatch=0.N Minimum ratio of bases that must remap. Default 0.95
       -gff  File is in gff/gtf format.  Note that the gff lines are converted
             separately.  It would be good to have a separate check after this
             that the lines that make up a gene model still make a plausible gene
             after liftOver
       -genePred - File is in genePred format
       -sample - File is in sample format
       -bedPlus=N - File is bed N+ format (i.e. first N fields conform to bed format)
       -positions - File is in browser "position" format
       -hasBin - File has bin value (used only with -bedPlus)
       -tab - Separate by tabs rather than space (used only with -bedPlus)
       -pslT - File is in psl format, map target side only
       -ends=N - Lift the first and last N bases of each record and combine the
                 result. This is useful for lifting large regions like BAC end pairs.
       -minBlocks=0.N Minimum ratio of alignment blocks or exons that must map
                      (default 1.00)
       -fudgeThick    (bed 12 or 12+ only) If thickStart/thickEnd is not mapped,
                      use the closest mapped base.  Recommended if using 
                      -minBlocks.
       -multiple               Allow multiple output regions
       -noSerial               In -multiple mode, do not put a serial number in the 5th BED column
       -minChainT, -minChainQ  Minimum chain size in target/query, when mapping
                               to multiple output regions (default 0, 0)
       -minSizeT               deprecated synonym for -minChainT (ENCODE compat.)
       -minSizeQ               Min matching region size in query with -multiple.
       -chainTable             Used with -multiple, format is db.tablename,
                                   to extend chains from net (preserves dups)
       -errorHelp              Explain error messages

그냥 실행하면 위와 같은 readme 를 띄워주고.. 

핵심은  `liftOver oldFile map.chain newFile unMapped` 이렇게 실행하면 된다는 것. 

이 글을 써야지 결심한 것은 여기에 들어가는 map.chain 파일이 매번 헷갈려서 이다. 다른 파일들은 간단한데, 인풋으로 넣어주는 oldFile은 변환시키고자 하는 데이터(작년 인강교재의 수특 주석)이고, newFile 에는 새로 만들고자 하는 파일 이름을 적으면 된다. unMapped 부분에는 liftover 되지 않은 것들을 저장할 파일 이름을 넣으면 되고. 

chain 파일이 위에서 잠깐 설명한 alignment 파일인데, 두 개 유전체를 pairwise 하게 비교한 파일이고, 두 종은 각각 reference, target으로 여겨진다. 그리고 매번 헷갈리는 부분은 이 chain file 이 어떤 데이터를 각각 reference, target 으로 상정하고 만들어진 chain 이어야 하는지!? 

조금 생각해보면  A → B 버전으로 옮겨가는 상황이니 reference:A → target:B 로 만들어진 파일을 사용하면 될 것 같은데 왜 매번 헷갈리는지 모를 일이다. 아무튼 여기에 활용되는 chain 파일은 출발 유전체를 reference, 도착 유전체를 target 으로 상정하고 만들어진 alignment 를 활용하면 된다.

---

이렇게 liftover 을 한 뒤에 서로 다른 버전 상에서 만들어져있는 정보를 비교할 수 있는 버전업-된 데이터를 얻을 수 있다. 실제 연구하는 데에서는
 - 서로 다른 종에 존재하는 유전자나 변이 정보를 비교해서 공통적으로 존재하는 유전자/변이를 찾는 데 활용할 수 있다(liftover tool 자체는 서로 다른 종에 대해 실행하는 것은 권장되지 않는다. 많은 정보가 손실될 수 있다는 우려에 의해..).
 - 새로 만든 assembly 상에서 변이정보를 밝힌 뒤 해당 변이가 어떤 유전자 주변에 있는 지 파악하기 위해 기존에 밝혀진 유전자 정보를 활용하고자 할 때 기존에 알려져있는 참조유전체 상의 유전자 위치를 새로 만든 assembly position 상으로 liftover 하여 사용할 수 있다.
