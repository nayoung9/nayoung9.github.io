---
layout: post
title: R을 활용해 아미노산 지도 그리기(aamap, amino-acid map)
---
## 단백질을 구성하는 아미노산 시퀀스의 패턴 관찰하기 
단백질은 아미노산의 연결로 만들어진 큰 분자로 생물체 내에서 다양한 기능을 수행한다. 단백질은 어떤 특성을 지니는 아미노산들이 어떤 3차 구조를 형성하여 연결되어있는지에 따라 그 기능과 특성이 달라진다. 따라서 밝혀진 단백질의 시퀀스를 바탕으로 그 3차구조, 기능 등을 예측할 수 있다.

본 포스팅에서는 R을 활용해 protein의 aminoacid sequence내에서 특정 amino-acid가 가지는 특이적 패턴을 발견하는 데 활용될 수 있는aamap을 그려보고자 한다.

레퍼런스로 활용한 논문은 [Coactivator condensation at super-enhancers links phase separation and gene control][paper]라는 제목을 가지고있는데, 이 논문의 figure중에 다음과 같은 그림이 있다. 

![med1](/assets/images/MED1.png)

이런 그림을 그리고자 했다. (옆 연구실 친구의 부탁으로 위와 같은 그림을 그려주는 코드가 있는지 알아보는데, 고전 사이트로 보이는 [Guide to Human Genome][refsite]에서 유사한 이미지를 `aamap`이라는 이름으로 제공하고 있었지만 위 레퍼런스 논문의 메쏘드나, 고전 사이트 어디에서도-물론 구글링도 한참 했다- 이런 그림을 그려주는 코드를 찾을 수 없었다.)

ggplot2패키지에서 히트맵(heat-map)을 그리는 용도로 사용되는 geom_tile()을 활용해 구현해 보았다. 이쪽 분야에서 heatmap은 주로 발현 정도(..)를 비롯한 연속적인 수치를 나타내는 데 사용되는데, 특정 시퀀스의 존재 여부를 신호로 생각하고 heatmap을 활용해 보았다.

프로그램의 흐름은 대략 다음과 같다.
1. 단백질 시퀀스의 아미노산을 하나씩 읽는다.
2. 읽어온 아미노산의 종류, 시퀀스 내에서의 위치를 저장한다.(heatmap 그리는 도구를 활용하므로 해당 위치의 value를 지정해 주는데 시퀀스 상에 존재하기만 하면 1, 그렇지 않으면 0 인 상황이니까 모두 동일한 값으로 처리했다.)
3. 개별적으로 만들어진 세 가지 벡터를 하나의 데이터프레임으로 묶는다.
4. (3)에서 만든 데이터프레임을 ggplot을 활용, heatmap으로 그려준다.
5. reference와 유사하게(단백질 시퀀스가 가지는 아미노산 패턴을 보다 잘 확인할 수 있도록 불필요한 요소들을 빼는 작업) 보이도록 plot의 세부 사항들을 조절한다. 


한 단계씩 코드와 함께 살펴보자!
1. 단백질 시퀀스의 아미노산을 하나씩 읽고, 아미노산의 종류, 시퀀스 내에서의 위치를 저장한다.
``` R
library(ggplot2)
seq = "MSSLLERLHAKFNQNRPWSETIKLVRQVMEKRVVMSSGGHQHLVSCLETLQKALKVTSLPAMTDRLESIARQNGLGSHLSASGTECYITSDMFYVEVQLDPAGQLCDVKVAHHGENPVSCPELVQQLREKNFEEFSKHLKGLVNLYNLPGDNKLKTKMYLALQSLEQDLSKMAIMYWKATNAAPLDKILHGSVGYLTPRSGGHLMNMKYYASPSDLLDDKTASPIILHEKNVPRSLGMNASVTIEGTSAMYKLPIAPLIMGSHPADNKWTPSFSAVTSANSVDLPACFFLKFPQPIPVSKAFVQKLQNCTGIPLFETPPTYLPLYELITQFELSKDPDPLPLNHNMRFYAALPGQQHCYFLNKDAPLPDGQSLQGTLVSKITFQHPGRVPLILNMIRHQVAYNTLIGSCVKRTILKEDSPGLLQFEVCPLSESRFSVSFQHPVNDSLVCVVMDVQDSTHVSCKLYKGLSDALICTDDFIAKVVQRCMSIPVTMRAIRRKAETIQADTPALSLIAETVEDMVKKNLPPASSPGYGMTTGNNPMSGTTTPTNTFPGGPITTLFNMSMSIKDRHESVGHGEDFSKVSQNPILTSLLQITGNGGSTIGSSPTPPHHTPPPVSSMAGNTKNHPMLMNLLKDNPAQDFSTLYGSSPLERQNSSSGSPRMEMCSGSNKAKKKKSSRVPPDKPKHQTEDDFQRELFSMDVDSQNPMFDVSMTADALDTPHITPAPSQCSTPPATYPQPVSHPQPSIQRMVRLSSSDSIGPDVTDILSDIAEEASKLPSTSDDCPPIGTPVRDSSSSGHSQSALFDSDVFQTNNNENPYTDPADLIADAAGSPNSDSPTNHFFPDGVDFNPDLLNSQSQSGFGEEYFDESSQSGDNDDFKGFASQALNTLGMPMLGGDNGEPKFKGSSQADTVDFSIISVAGKALGAADLMEHHSGSQSPLLTTGELGKEKTQKRVKEGNGTGASSGSGPGSDSKPGKRSRTPSNDGKSKDKPPKRKKADTEGKSPSHSSSNRPFTPPTSTGGSKSPGSSGRSQTPPGVATPPIPKITIQIPKGTVMVGKPSSHSQYTSSGSVSSSGSKSHHSHSSSSSSLASASTSGKVKSSKSEGSSSSKLSGSMYASQGSSGSSQSKNSSQTGGKPGSSPITKHGLSSGSSSTKMKPQGKPSSLMNPSISKPNISPSHSRPPGGSDKLASPMKPVPGTPPSSKAKSPISSGSSGSHVSGTSSSSGMKSSSGSASSGSVSQKTPPASNSCTPSSSSFSSSGSSMSSSQNQHGSSKGKSPSRNKKPSLTAVIDKLKHGVVTSGPGGEDPIDSQMGASTNSSNHPMSSKHNTSGGEFQSKREKSDKDKSKVSASGGSVDSSKKTSESKNVGSTGVAKIIISKHDGGSPSIKAKVTLQKPGESGGDGLRPQIASSKNYGSPLISGSTPKHERGSPSHSKSPAYTPQNVDSESESGSSIAERSYQNSPSSEDGIRPLPEYSTEKHKKHKKEKKKVRDKDRDKKKSHSMKPENWSKSPISSDPTASVTNNPILSADRPSRLSPDFMIGEEDDDLMDVALIGN"
position <- c()
amino_acids <- c()
on_off <- c()
for (i in 1:nchar(seq)){
  aa= substr(seq,i,i)
  amino_acids <- c(amino_acids, aa)
  position <- c(position, as.integer(i))
  on_off <- c(on_off, 1)
}
```

2. 개별적으로 만들어진 세 가지 벡터를 하나의 데이터프레임으로 묶는다.
``` R
df<-data.frame() # initialize data frame
df<- data.frame(position, amino_acids, on_off)
```

3. (3)에서 만든 데이터프레임을 ggplot을 활용, heatmap으로 그려준다.
``` R
p<- ggplot(df, aes(x=position, y=amino_acids, fill=on_off))
p<- p+geom_tile(fill="black")
```
`fill=block` 옵션을 주긴 했는데, 해당 옵션을 주지 않았다고 생각하고 그림을 먼저 확인해보면 다음과 같다.
![med_v1](/assets/images/MED_v1.png)


4. ggplot의 세부 옵션들을 조정해 그림을 다듬는다.
``` R
p<-p+scale_x_discrete(expand=c(0,0))
p<-p+scale_y_discrete(position="right")
p<-p+theme(legend.position = "none",panel.background = element_blank(), panel.grid.major = element_blank(), panel.border=element_rect(color="black", fill=NA), axis.title.y=element_blank())
p <- p+theme(axis.text.y=element_text(color=ifelse(levels(df$amino_acids)=="L","red",ifelse(levels(df$amino_acids)=="K","red",ifelse(levels(df$amino_acids)=="G","red",ifelse(levels(df$amino_acids)=="K","red",ifelse(levels(df$amino_acids)=="S","red","black"))))), face=ifelse(levels(df$amino_acids)=="D","bold",ifelse(levels(df$amino_acids)=="L","bold",ifelse(levels(df$amino_acids)=="G","bold",ifelse(levels(df$amino_acids)=="S","bold",ifelse(levels(df$amino_acids)=="K","bold","plain")))))))
```
- x축이 시퀀스의 위치를 나타내므로 0과 마지막 베이스 위치 바깥의 부분을 제거했다.
- 기본적으로 plot의 좌측에 위치하는 y축을 plot우측으로 옮겨주었다.
- plot의 배경색, 레전드, plot테두리 등의 설정을 변경했다.
- y축 위의 특정 아미노산을 강조하기 위해 `theme(axis.text.y  = element_text(color =, face = ))` 옵션 안에 `ifelse(조건, true일 때 실행, false일 때 실행)`함수를 활용했다. 여러 개의 아미노산을 강조해야 할 경우 ifelse 함수의 마지막 인자에 새로운 ifelse함수를 넣어주어 else조건에서 새 조건을 확인할 수 있도록 했다. 조건을 줄이고 싶은 경우 ifelse문 전체를 지우고 face, color 옵션에 각각 기본값인 `"plain"`, `"black"`을 넣어주면 된다.


모든 조건을 조절해 주었을 때 최종적으로 완성된 그림은 다음과 같다.
![med_v2](/assets/images/MED_v2.png)

위 그림을 통해 레퍼런스와 동일한 MED1 단백질의 아미노산 패턴인데 특정 지역에 S(Serin, 세린)아미노산이 특이하게 가까이, 또 많이 분포하고 있는 것을 알 수 있는데, 특정 지역을 높은 밀도로 점유하고 있는 저 아미노산의 특성 덕분에 저 지역을 IDR이라고 추정할 수 있다고 했다. 자세한 것은 위의 레퍼런스 논문이나 [위키피디아][wiki], 혹은 [관련 논문][related_paper]를 참고할 수 있겠다.

만든 plot 저장은 아래과 같은 식으로 저장할 수 있고, 바로 출력해서 Rstudio에서 직접 export할 수 있다.
``` R
ggsave("example.png", plot=last_plot())
```
[paper]: http://dx.doi.org/10.1126/science.aar3958
[refsite]: http://www.cshlp.org/ghg5_db/recinfo/224/22415.shtml
[wiki]: https://en.wikipedia.org/wiki/Intrinsically_disordered_proteins
[related_paper]: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3949125/


아래는 전체 코드 
``` R
library(ggplot2)

seq = "MSSLLERLHAKFNQNRPWSETIKLVRQVMEKRVVMSSGGHQHLVSCLETLQKALKVTSLPAMTDRLESIARQNGLGSHLSASGTECYITSDMFYVEVQLDPAGQLCDVKVAHHGENPVSCPELVQQLREKNFEEFSKHLKGLVNLYNLPGDNKLKTKMYLALQSLEQDLSKMAIMYWKATNAAPLDKILHGSVGYLTPRSGGHLMNMKYYASPSDLLDDKTASPIILHEKNVPRSLGMNASVTIEGTSAMYKLPIAPLIMGSHPADNKWTPSFSAVTSANSVDLPACFFLKFPQPIPVSKAFVQKLQNCTGIPLFETPPTYLPLYELITQFELSKDPDPLPLNHNMRFYAALPGQQHCYFLNKDAPLPDGQSLQGTLVSKITFQHPGRVPLILNMIRHQVAYNTLIGSCVKRTILKEDSPGLLQFEVCPLSESRFSVSFQHPVNDSLVCVVMDVQDSTHVSCKLYKGLSDALICTDDFIAKVVQRCMSIPVTMRAIRRKAETIQADTPALSLIAETVEDMVKKNLPPASSPGYGMTTGNNPMSGTTTPTNTFPGGPITTLFNMSMSIKDRHESVGHGEDFSKVSQNPILTSLLQITGNGGSTIGSSPTPPHHTPPPVSSMAGNTKNHPMLMNLLKDNPAQDFSTLYGSSPLERQNSSSGSPRMEMCSGSNKAKKKKSSRVPPDKPKHQTEDDFQRELFSMDVDSQNPMFDVSMTADALDTPHITPAPSQCSTPPATYPQPVSHPQPSIQRMVRLSSSDSIGPDVTDILSDIAEEASKLPSTSDDCPPIGTPVRDSSSSGHSQSALFDSDVFQTNNNENPYTDPADLIADAAGSPNSDSPTNHFFPDGVDFNPDLLNSQSQSGFGEEYFDESSQSGDNDDFKGFASQALNTLGMPMLGGDNGEPKFKGSSQADTVDFSIISVAGKALGAADLMEHHSGSQSPLLTTGELGKEKTQKRVKEGNGTGASSGSGPGSDSKPGKRSRTPSNDGKSKDKPPKRKKADTEGKSPSHSSSNRPFTPPTSTGGSKSPGSSGRSQTPPGVATPPIPKITIQIPKGTVMVGKPSSHSQYTSSGSVSSSGSKSHHSHSSSSSSLASASTSGKVKSSKSEGSSSSKLSGSMYASQGSSGSSQSKNSSQTGGKPGSSPITKHGLSSGSSSTKMKPQGKPSSLMNPSISKPNISPSHSRPPGGSDKLASPMKPVPGTPPSSKAKSPISSGSSGSHVSGTSSSSGMKSSSGSASSGSVSQKTPPASNSCTPSSSSFSSSGSSMSSSQNQHGSSKGKSPSRNKKPSLTAVIDKLKHGVVTSGPGGEDPIDSQMGASTNSSNHPMSSKHNTSGGEFQSKREKSDKDKSKVSASGGSVDSSKKTSESKNVGSTGVAKIIISKHDGGSPSIKAKVTLQKPGESGGDGLRPQIASSKNYGSPLISGSTPKHERGSPSHSKSPAYTPQNVDSESESGSSIAERSYQNSPSSEDGIRPLPEYSTEKHKKHKKEKKKVRDKDRDKKKSHSMKPENWSKSPISSDPTASVTNNPILSADRPSRLSPDFMIGEEDDDLMDVALIGN"

position <- c()
amino_acids <- c()
on_off <- c()

for (i in 1:nchar(seq)){
  aa= substr(seq,i,i)
  amino_acids <- c(amino_acids, aa)
  position <- c(position, as.integer(i))
  on_off <- c(on_off, 1)
}

df<-data.frame() # initialize data frame
df<- data.frame(position, amino_acids, on_off)

p<- ggplot(df, aes(x=position, y=amino_acids, fill=on_off))
p<- p+geom_tile(fill="black")
p<-p+scale_x_discrete(expand=c(0,0))
p<-p+scale_y_discrete(position="right")
p<-p+theme(legend.position = "none",panel.background = element_blank(), panel.grid.major = element_blank(), panel.border=element_rect(color="black", fill=NA), axis.title.y=element_blank())
p <- p+theme(axis.text.y=element_text(color=ifelse(levels(df$amino_acids)=="L","red",ifelse(levels(df$amino_acids)=="K","red",ifelse(levels(df$amino_acids)=="G","red",ifelse(levels(df$amino_acids)=="K","red",ifelse(levels(df$amino_acids)=="S","red","black"))))), face=ifelse(levels(df$amino_acids)=="D","bold",ifelse(levels(df$amino_acids)=="L","bold",ifelse(levels(df$amino_acids)=="G","bold",ifelse(levels(df$amino_acids)=="S","bold",ifelse(levels(df$amino_acids)=="K","bold","plain")))))))

ggsave("example.png", plot=last_plot()))
```
