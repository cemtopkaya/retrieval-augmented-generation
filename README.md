# Retrieval-Augmented Generation (RAG)
Benim anlaıdğım şekliyle RAG belgelendirmesi.

[RAG (Yeniden alarak - Artırılmış Üretim)](https://newsletter.nocode.ai/p/guide-retrieval-augmented-generation), Büyük Dil Modellerine (**L**arge **L**anguage **M**odels) harici kaynaklardan ilgili bilgileri besleyerek yardımcı olur. 
Bu, LLM'lerin yanıtları oluştururken bu bağlamı dikkate almasına olanak tanıyarak yanıtları daha doğru ve güncel hale getirir.

![image](https://github.com/cemtopkaya/retrieval-augmented-generation/assets/261946/d277a72a-2e32-4a20-baf6-00bde5a40299)

- Belgeyi Yükle: Belgeyi veya veri kaynağını yükleyerek başlayın. 
- Parçalara Böl: Belgeyi yönetilebilir parçalara bölün.
- Gömmeler Oluşturun: Bu parçaları, gömmeleri kullanarak vektör temsillerine dönüştürün.
- Vektör Veritabanında Saklayın: Verimli bir şekilde geri almak için bu vektörleri veritabanına kaydedin.
- Kullanıcı Etkileşimi: Kullanıcıdan sorgular veya girdiler alın ve bunları yerleştirmelere dönüştürün.
- VectorDB'de Anlamsal Arama: Kullanıcının sorgusuna dayalı olarak anlamlı bir arama gerçekleştirmek için vektör veritabanına bağlanın.
- Yanıtların Alınması ve İşlenmesi: İlgili yanıtları alın, bunları bir LLM'den geçirin ve bir yanıt oluşturun.
- Kullanıcıya Yanıtın İletilmesi: LLM tarafından oluşturulan son çıktıyı kullanıcıya geri sunun.

## LangChain
RAG ile yapmak istediğimiz işleri kolaylaştıran hattın adına LangChain diyoruz. LLM'ler etrafında oluşturulmuş açık kaynaklı bir çerçeve olan LangChain, sohbet robotları, özetleme ve çok daha fazlası dahil olmak üzere çeşitli GenAI uygulamalarının tasarımını ve geliştirilmesini kolaylaştırır. Kütüphanenin temel fikri, karmaşık yapay zeka görevlerini basitleştirmek ve Yüksek Lisans (LLM) çevresinde daha gelişmiş kullanım senaryoları oluşturmak için farklı bileşenleri bir araya "zincirlemek"tir.

![image](https://github.com/cemtopkaya/retrieval-augmented-generation/assets/261946/4c21d8ba-4d7b-4e2d-bfd9-0328a64257c9)

## LangFlow
LangFlow, LangChain için özel olarak tasarlanmış bir web aracıdır. Kullanıcıların herhangi bir kodlamaya gerek kalmadan LangChain uygulamalarını oluşturmak ve test etmek için bileşenleri basitçe sürükleyip bırakabilecekleri bir kullanıcı arayüzü sunar. Ancak yapay zeka uygulama akışınızı tasarlamak amacıyla LangFlow'u kullanabilmek için öncelikle LangChain'in nasıl çalıştığı ve farklı bileşenleri hakkında temel bir anlayışa sahip olmanız gerekir.

## Ollama
`Ollama` açık kaynaklı LLM'leri kurup çalıştırmanın en iyi ve aynı zamanda en kolay yoludur. Diğerlerinin yanı sıra `Llama 2` ve `Mistral` gibi en yetenekli LLM'leri destekler ve mevcut modellerin listesini [ollama.ai/library](https://ollama.com/library) adresinde bulabilirsiniz.

### Ollama'yı Kurmak 
Öncelikle [Ollama indirme sayfasına](https://ollama.com/download) gidin, işletim sisteminize uygun sürümü seçin, indirip yükleyin. 

Linux makinede:
```sh
curl -fsSL https://ollama.com/install.sh | sh
```

![Ollamayı Kurmak](images/install_ollama.png)

Ollamanın komutları basitçe bize  bize anlatıyor:

```sh
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
```

Ollama kuruluyken terminalinizi açın ve aşağıdaki komutları girin. 

```sh
ollama serve
ollama pull llama2
ollama pull nomic-embed-text
ollama run llama2
```

Bu komutlar; 
- modelleri indirecek
- ve makinenizde yerel olarak modelleriniz çalıştıracaktır.

Bu proje için, Büyük Dil Modelimiz (LLM) olarak `Llama2`'yi ve gömme modeli için `nomic-embed-text`'i kullanacağız. 

> `Nomic-embed-text`, geniş bağlam penceresine sahip güçlü bir açık kaynaklı gömme modelidir.

Bu, herhangi bir bulut hizmetine ihtiyaç duymadan uygulamanın tamamını yerel olarak çalıştırmamızı sağlar!

> Ollama hizmetinin 11434 portunda çalıştığını `netstat -pltn` ile göreceksiniz:
> 
> ![Ollama port on 11434](images/ollama_port_11434.png)

> llama2'yi indirdiğimizde 3.8GB büyüklüğünde bir modeli çekmiş olacağız:
> 
> ![llama2 indirmek](images/download_llama2.png)

> nomic-embed-text gömme modelini indirdiğimizde 

![nomic-embed-text indirmek](images/download_nomic-embed-text.png) 

#### Mistral'ı Kurmak 

Mistral'ı indirip ollamayı çalıştırarak doğrudan yazışma ekranını açacaktır

```sh
ollama run mistral
```

![mistralı kurmak](images/install_mistral.png)

#### Yüklü Modelleri Listelemek (ollama ls)
`ollama ls` komutuyla yüklü modelleri listeleyebiliriz:

![yüklü modelleri listele](images/list_models.png)

### LangFlow'u Ayarlama 

LangFlow'a başlamadan önce Python'un bilgisayarınızda kurulu olup olmadığını kontrol etmeniz önemlidir. Python sürümünüz 3.9'un üzerinde ancak 3.12'den eski olmalıdır. 
Şimdi LangFlow'u kurmaya geçelim. Bunu sanal ortamda yapmanızı öneririm. Bu yaklaşım, bağımlılıkların kendi alanı içerisinde düzgün bir şekilde yönetilmesine yardımcı olur. Mac bilgisayarımda işleri ayarlamak için Conda'yı kullanıyorum. Python 3.11 ile "langflow" adında bir sanal ortam oluşturmak için aşağıdaki komutları bir komut satırı terminaline girmeniz yeterlidir.

```shell
conda create -n langflow python=3.11
conda activate langflow
pip install langflow
```

Conda'nız yoksa aşağıdaki komutlarla doğrudan Python ile sanal bir ortam da kurabilirsiniz.
```shell
python -m venv langflow
source langflow/bin/activate
pip install langflow
```

Kurulumu tamamladıktan sonra LangFlow'u başlatmak, terminalinize "langflow run" girmek kadar basittir.
