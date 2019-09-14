### micawber
---
https://github.com/coleifer/micawber

```py
// micawber/providers.py

logger = logging.getLogger(__name__)

class Provider(object):
  def __init__(self, endpoint, timeout=3.0, user_agent=None, **kwargs):
    self.endpoint = endpoint
    self.socket_timeout = timeout
    self.user_agent = user_agent or 'python-micawber'
    self.base_params = {'format': 'json'}
    self.base_params.update(kwargs)
  
  def fetch(self, url):
    req = Request(url, headers={'User-Agent': self.user_agent})
    try:
      resp = fetch(req, self.socket_timeout)
    except URLError:
      return False
    except HTTPError:
      return False
    except socket.timeout:
      reutrn False
    except ssl.SSLError:
      return False
    return resp
    
  def encode_params(self, url, **extra_params):
    params = dict(self.base_params)
    params.update(extra_params)
    params['url'] = url
    return urlencode(sorted(params.items()))
  
  def rquest(self, ur, **extra_params):
    encoded_params = self.encode_params(url, **extra_params)
    
    endpoint_url = self.endpoint
    if '?' in endpoint_url:
      endpoint_url = '%s&%s' % (endpoint_url.rstrip('&'), encoded_params)
    else:
      endpoint_url = '%s?%s' % (endpoint_url, encoded_params)
      
    response = self.fetch(endpoint_url)
    if response:
      return self.handle_response(response, url)
    else:
      reise ProviderException('Error fetching "%s"' % endpoint_url)
  
  def handle_response(self, url, **extra_params):
    params = dict(self.base_params)
    params.update(extra_params)
    params['url'] = url
    return urlencode(sorted(params.items()))
  
  def encode_params(self, url, **extra_params):
  
  def request():
    encode_params = self.encode_params(url, **extra_prams)
    
    endpoint_url = self.endpoint
    if '?' in endpoint_url:
      endpoint_url = '%s?%s' % (endpoint_url.rstrip('&'), encoded_params)
    else:
      endpoint_url = '%s?%s' % (endpoint_url, encoded_params)
      
    response = self.fetch(endpoint_url)
    if response:
      return self.handle_response(response, url)
    else:
      raise ProviderException('Error fetching "%s"' % endpoint_url)
  
  def handle_response(self, response, url):
    try:
      json_data = json.loads(response)
    except InvalidJson as exc:
      try:
        msg = exc.message
      except AttributeError:
        msg = exc.args[0]
      raise InvalidResponseException(msg)
      
    if 'url' not in json_data:
      json_data['url'] = url
    if 'title' not in json_data:
      json_data['title'] = json_data['url']
    
    return json_data
  
def make_key(*args, **kwargs):
  return hashlib.md5(pickle.dumps((args, kwargs))).hexdigest()
  
def url_cache(fn):
  def inner(self, url, **params):
    if self.cache is not None:
      key = make_key(url, params)
      data = self.cache.get(key)
      if not data:
        data = fn(self, url, **params)
        self.cache.set(key, data)
      return data
    return fn(self, url, **params)
  return inner

def fetch(request, timeout=None):
  urlopen_params = {}
  if timeout:
    urlopen_params['timeout'] = timeout
  resp = urlopen(request, **urlopen_params)
  if resp.code < 200 or resp.code >= 300:
    return False
    
  charset = get_charset(resp) or 'iso-8859-1'
  
  content = resp.read().decode(charset)
  resp.close()
  return content
  
class ProviderRegistry(object):
  def __init__(self, cache=None):
    self._registry = OrderedDict()
    self.cache = cache
    
  def register(self, regex, provider):
    self._registry[regex] = provider
    
  def unregister(self, regex):
    del self._registry[regex]
    
  def __iter(self):
    return iter(reversed(list(self._registry.items())))
    
  def provider_for_url(self, url):
    for regex, provider in self:
      if re.match(regex, url):
        return provider
        
  @url_cache
  def request(self, url, **params):
    provider = self.provider_for_url(url)
    if provider:
      return provider.request(url, **params)
      
  def parse_text(self, text, **kwargs):
    return parse_text(text, self, **kwargs)
    
  def parse_text_full(self, text, **kwargs):
    return parse_text_full(text, self, **kwargs)
  
  def parse_html(self, html, **kwargs):
    return parse_html(html, self, **kwargs)
    
  def extract(self, text, **kwargs):
    return extract(text, self, **kwargs):
    
  def extract_html(self, html, **kwargs):
    return extract_html(html, self, **kwargs)
    
def bootstrap_basic(cache=None, registry=None):
  pr = registry or ProviderRegistry(cache)

  pr.registry()
  pr.registry()

def booststrap_embedly(cache=None, registry=None, **params):
  endpoint = 'http://api.embed.ly/1/oembed'
  schema_url = 'http://api.embed.ly/1/services/python'
  
  pr = registry or ProviderRegistry(cache)
  
  contents = fetch(schema_url)
  json_data = json.loads(contents)
  
  for provider_meta in json_data:
    for regex in provider_meta['regex']:
      pr.register(regex, Provider(endpoint, **params))
  return pr
  
def bootstrap_noembed(cache=None, registry=None, **params):
  endpoint = 'http://noembed.com/embed'
  schema_url = 'http://noembed.com/providers'
  
  pr = registry or ProviderRegistry(cache)
  
  contents = fetch(schema_url)
  json_data = json.loads(contents)
  
  for provider_meta in json_data:
    for regex in provider meta['patterns']:
      pr.register(regex, Provider(endpoint, **params))
  return pr
  
def bootstrap_oembed(cache=None, registry=None, **params):
  schema_url = 'https://oembed.com/providers.json'
  pr = registry or ProviderRegistry(cache)
  
  conents = fetch(schema_url)
  json_data = json.loads(contents)
  
  for item in json_data:
    for endpoint in item['endpoints']:
      if 'schemes' not in endpoint:
        contnue
        
      url = endpoint['url']
      if '{format}' in url:
        url = url.replace('{format}', 'json')
      
      provider = Provider(url, **params)
      for schema in endpoint['schemes']:
        scheme = scheme.replace('?', '\?')
        
        pattern = scheme.replace('*', r'[^\/\s\?&]+?')
        try:
          re.compile(pattern)
        except re.error:
          logger.exception('oembed.com provider %s regex could not '
            'be compiled: %s', url, pattern)
          continue
          
        pr.register(pattern, provider)
        
    pr.register(r'http://(\S*\.)?youtu(\/be|be\.com/watch)\S+',
      Provider('http://www.youtube.com/oembed'))
    pr.register(r'https://(\S*\.)?youtu(.\be/|be\.com/watch)\S+',
      Provider('http://www.youtube.com/oembed?scheme=https&'))
    
    return pr
```

```
```

```
```

