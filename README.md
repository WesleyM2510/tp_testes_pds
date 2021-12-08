**Nome:** Wesley Maciel

**Matrícula:** 2017086031

**Projeto:** Boto3  (https://github.com/boto/boto3)


Boto3 é o Software Development Kit (SDK) da Amazon Web Services (AWS) para Python, que permite aos desenvolvedores a possibilidade de criar, deletar e gerenciar serviços em ambiente AWS. Os testes a serem apresentados abaixo, foram escritos através da bibliotéca unittest e são referentes à suite de testes presente [neste link](https://github.com/boto/boto3/blob/develop/tests/unit/test_boto3.py).

**Teste 1**
```python
import boto3

from tests import mock, unittest


class TestBoto3(unittest.TestCase):
    ...
    def setUp(self):
        self.session_patch = mock.patch('boto3.Session', autospec=True)
        self.Session = self.session_patch.start()
    
    def tearDown(self):
        boto3.DEFAULT_SESSION = None
        self.session_patch.stop()

    def test_create_default_session(self):
        session = self.Session.return_value

        boto3.setup_default_session()

        assert boto3.DEFAULT_SESSION == session

```
Os dois métodos iniciais são responsavei por executar antes e depois de cada método de teste presente na classe TestBoto3. O método setUp() prepara a fixture de teste, como é possível ver no código acima, mock.patch é utilizado para "mockar" a classe [Session](https://github.com/boto/boto3/blob/develop/boto3/session.py). A classe Session é responsvel por armazenar o estado de configuração que permite interagir com os recursos na AWS, ela armazena access key e secret key, que são chaves responsaveis por acesso as contas na AWS e questões como a região default que os recursos serão alocados. 

O método de teste test_create_default_session() fica responsavel por validar se após a chamada de [setup_default_session()](https://github.com/boto/boto3/blob/73c5c6cce01eb63cfde6bf52f2aec0a5b6c6a7af/boto3/__init__.py#L28) que instancia um objeto Session na variavel boto3.DEFAULT_SESSION, se o valor é igual ao da Session obtida através do mock.



**Teste 2**
```python

class TestBoto3(unittest.TestCase):
    ...
    @mock.patch('boto3.setup_default_session',
                wraps=boto3.setup_default_session)
    def test_client_creates_default_session(self, setup_session):
        boto3.DEFAULT_SESSION = None

        boto3.client('sqs')

        assert setup_session.called
        assert boto3.DEFAULT_SESSION.client.called
    ...
```
O método de teste acima usa o decorator mock.patch, para "mockar" o parâmetro setup_session e testa criação de default session através do boto3.client(). Os clients fornecem uma interface de baixo nível para a AWS, cujo os métodos mapeiam para as APIs da AWS. No exemplo acima é feita uma chamada no cliente para o recurso Simple Queue Service (SQS).

**Teste 3**
```python
class TestBoto3(unittest.TestCase):
    ...
    @mock.patch('boto3.setup_default_session',
                wraps=boto3.setup_default_session)
    def test_resource_creates_default_session(self, setup_session):
        boto3.DEFAULT_SESSION = None

        boto3.resource('sqs')

        assert setup_session.called
        assert boto3.DEFAULT_SESSION.resource.called
    ...
```
O funcionamento do mock acima é idêntico ao do **Teste 2**, entretanto este método de teste usa um boto3.resource() para criar a default session. Um resource funciona como uma forma de acessar os serviços AWS, com um nível mais alto de abstração, funcionando de forma orientada a objetos, trabalhar com os serviços na AWS através de resources acaba sendo mais intuitivo do que utilizar clients, entretanto os resources não possuem todos serviços disponíveis na AWS.

