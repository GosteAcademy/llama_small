## Как собирать Llama + OpenHermes

    sudo apt update
    sudo apt install make g++ git-lfs
    git lfs install

    git clone https://huggingface.co/teknium/OpenHermes-2.5-Mistral-7B
    git clone https://github.com/ggerganov/llama.cpp

(Если нет poetry - можно поставить так как написано: https://python-poetry.org/docs/)

    poetry new llama
    mv llama/pyproject.toml llama.cpp/
    rm -rf llama
    cd llama.cpp
    make

(здесь заглянуть в requirements и поставить все перечисленные там пакеты с указанными версиями)

    poetry add numpy==1.24.4
    poetry add sentencepiece==0.1.98
    poetry add "gguf>=0.1.0"

    poetry shell


    python convert.py <абсолютный путь>/OpenHermes-2.5-Mistral-7B/ --outfile <абсолютный путь>/OpenHermes-2.5-Mistral-7B/ggml-model-f16.gguf --outtype f16
(автор пишет создать символическую ссылку на OpenHermes-2.5-Mistral-7B и конвертировать с её использованием в директорию llama.cpp, но у меня так не сработало)

    ./quantize <абсолютный путь>/OpenHermes-2.5-Mistral-7B/ggml-model-f16.gguf <абсолютный путь>/OpenHermes-2.5-Mistral-7B/ggml-model-q8_0.gguf q8_0
    ./quantize <абсолютный путь>/OpenHermes-2.5-Mistral-7B/ggml-model-f16.gguf <абсолютный путь>/OpenHermes-2.5-Mistral-7B/ggml-model-q4_0.gguf q4_0

В документации репозитория llama.cpp есть запуск сервера. У меня не получилось - памяти видеокары не хватило нина одну модель. Поэтому для экспериментов использовал другой проект. Выкладывать не обязательно (только установить - см. ниже), интересна документация: https://github.com/abetlen/llama-cpp-python

    poetry add llama-cpp-python

Дальше открывем проект llama.cpp в Pycharm (или кто в чём может), прописываем интерпретатор из виртуального окружения, что создал poetry (посмотреть - например при выполнении команды poetry shell)

И пишем что-то вроде:

    from llama_cpp import Llama

    llm = Llama(
        # model_path="/home/vic/work/git/OpenHermes-2.5-Mistral-7B/ggml-model-q4_0.gguf"
        model_path="/home/vic/work/git/OpenHermes-2.5-Mistral-7B/ggml-model-q8_0.gguf"
        # model_path="/home/vic/work/git/OpenHermes-2.5-Mistral-7B/ggml-model-f16.gguf"
    )

    prompt = str(input('Введите вопрос: '))

    while prompt:

        output = llm(
              f"Q: {prompt} A: ",
              max_tokens=1000,
              stop=["Q:", "\n"], # Stop generating just before the model would generate a new question
              echo=True # Echo the prompt back in the output
        )

        results = [c.get('text', '') for c in output.get('choices', [])]

        if results:
            print('Ответ(ы):')
            print('\n'.join(results))
        else:
            print('Нет вариантов ответа')

        prompt = str(input('\n\nВведите вопрос: '))







