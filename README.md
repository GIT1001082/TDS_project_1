# GITHUB Scraping

 - The data was scraped using the github API and python requests. Post caching, the requst results were converted to json and then after some cleaning, saved to a csv file.
 - The most interesting facts about the data was that there were comparatively very few users who worked at big tech giants.
 - Based on the analysis, I would like to suggest developers to provide concise and accurate information in their bio instead of long streached out descriptions.


## Code

The requests library in python was used to fetch the data. The code is given below:


The code below was used to create a cache folder and implement cache for the fetched data.
```
if not os.path.exists('.cache'):
    os.makedirs(".cache")


def cache_data(url):
    filename = hashlib.md5(url.encode("utf-8")).hexdigest()
    path = os.path.join('.cache', filename)
    if not os.path.exists(path):
        # print('Fetching data from source')
        response = requests.get(url, headers=HEADERS)
        with open(path, 'w', encoding="utf-8") as f:
            # print('cacheing data')
            json.dump(response.json(), f)
    # print('fetching data from cache')
    with open(path, 'r', encoding="utf-8") as f:
        data = json.load(f)
    return data
```


The following functions were used to fetch user data and repository information for the users.
```
def get_users_by_location(location, min_followers=50):
    page = 1
    user_name = []
    while page < 10:
        url = f'https://api.github.com/search/users?q=location:{
            location}+followers:>{min_followers}&per_page=100&page={page}'
        response = cache_data(url)
        data = response  # .json()
        page += 1
        if len(data['items']) > 0:
            # print(f'page: {page}')
            for user in data['items']:
                user_name.append(user['login'])
        else:
            # print('Fetched user_names')
            break

    return user_name


def fetch_user_information(user):
    global users
    url = f'https://api.github.com/users/{user}'
    response = cache_data(url)
    data = response  # .json()
    i_user_data = []
    # print(f'Fetching user data for {data['login']}')
    user_data = {'login': data['login'],
                 'name': str(data['name']).title(),
                 'company': str(data['company']).strip().upper() if '@' not in str(data['company']) else str(data['company']).split('@')[1].strip().upper(),
                 'location': str(data['location']).title(),
                 'email': data['email'] if data['email'] is not None else "Not available",
                 'hireable': "Yes" if data['hireable'] else 'No',
                 'bio': str(data['bio']).strip(),
                 'public_repos': data['public_repos'],
                 'followers': data['followers'],
                 'following': data['following'],
                 'created_at': datetime.strptime(data['created_at'], '%Y-%m-%dT%H:%M:%SZ').date()}
    users.append(user_data)
    i_user_data.append(user_data)
    return i_user_data


def fetch_repo_information(user):
    global repositories
    url = f'https://api.github.com/users/{user}/repos'
    response = cache_data(url)
    data = response  # .json()
    repo_count = 0
    user_repo = []
    if len(data) > 500:
        while repo_count <= 500:
            for item in data:
                # print(f'Fetching repo data for {item['full_name']}')
                repo = {'login': item['owner']['login'],
                        'full_name': item['full_name'],
                        'created_at': datetime.strptime(item['created_at'], '%Y-%m-%dT%H:%M:%SZ').date(),
                        'stargazers_count': item['stargazers_count'],
                        'watchers_count': item['watchers_count'],
                        'language': str(item['language']).title(),
                        'has_projects':  str(item['has_projects']).title(),
                        'has_wiki': item['has_wiki'],
                        'license_name': item['license']['name'] if item['license'] else "Not Available"}
                repositories.append(repo)
                user_repo.append(repo)
                repo_count += 1
    else:
        for item in data:
            # print(f'Fetching repo data for {item['full_name']}')
            repo = {'login': item['owner']['login'],
                    'full_name': item['full_name'],
                    'created_at': datetime.strptime(item['created_at'], '%Y-%m-%dT%H:%M:%SZ').date(),
                    'stargazers_count': item['stargazers_count'],
                    'watchers_count': item['watchers_count'],
                    'language': item['language'],
                    'has_projects':  str(item['has_projects']).title(),
                    'has_wiki': item['has_wiki'],
                    'license_name': item['license']['name'] if item['license'] else "Not Available"}
            repositories.append(repo)
            user_repo.append(repo)
    return user_repo
```


The following function was used to complile the data into csv files:
```
def compile_information():
    global users, repositories
    user_name = get_users_by_location('mumbai', min_followers=50)
    for user in user_name:
        fetch_user_information(user)
        # print('fetched user data')
        fetch_repo_information(user)
        # print('fetched repo data')

    with open('users.csv', 'w', newline='') as userfile:
        writer = csv.DictWriter(userfile, fieldnames=['login', 'name', 'company', 'location',
                                'email', 'hireable', 'bio', 'public_repos', 'followers', 'following', 'created_at'])
        writer.writeheader()
        writer.writerows(users)

    with open('repositories.csv', 'w') as repofile:
        writer = csv.DictWriter(repofile, fieldnames=[
                                'login', 'full_name', 'created_at', 'stargazers_count', 'watchers_count', 'language', 'has_projects', 'has_wiki', 'license_name'])
        writer.writeheader()
        writer.writerows(repositories)
```