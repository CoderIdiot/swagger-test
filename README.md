# swagger-test
autogenerate mocha-tests from a swagger spec

## Usage
```bash
docker pull luckymark/swagger-test
docker run -it -v ${PWD}/apitest:/home/apitest luckymark/swagger-test node ./dist/src/app.js -a ${url-of-spec} -s ${url-root-of-api-server} -f ${localfile}

ls ./apitest
```
then you can run test:
```bash
docker run -it -v ${PWD}/apitest:/home/apitest luckymark/swagger-test npm run apitest
```

## how to write your test in a spec.yaml:
```yaml
openapi: 3.0.0
info:
  description: Example API
  version: 0.0.1
#use a extention tag x-test on top level to declare some fixtures:
x-test:
# prop:['before']will be put to the front of every test files.
    before: |
        let token

        async function requestToken() {
            let res = await request.post("/user/login")
                .type('json')
                .send({
                    name: "Andy",
                    password: "12345678"
                })
            assert.equal(res.statusCode, 200)
            token = res.body.token
        }

# prop:['beforeEach']will be put to the front of every testSuite.   
    beforeEach: |
        static async  before() { await requestToken() }
paths:
  /user:
    get:
      operationId: userList
      description: "get user list"
      parameters:
        - in: query
          name: pageIndex
          schema:
            type: integer
            format: int32
        - in: query
          name: pageSize
          schema:
            type: integer
            format: int32
      responses:
        '200':
          description: return user list
          content:
            application/json:
              schema:
                allOf:
                  - $ref: '#/components/schemas/Page'
                  - type: object
                    properties:
                      data:
                        $ref: '#/components/schemas/UserList'
# x-test at here declare a single testcase:
          x-test:
            # 'getUserList' is the name of this testcase:  
            getUserList:
              parameters:
                page_index: 1
                page_size: 20

  '/user/{id}':
    put:
      operationId: editUser
      description: edit an user object
      parameters:
        - name: id
          in: path
          description: user_id
          required: true
          schema:
            type: integer
            format: int64
      responses:
        '200':
          description: return an user object
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
          x-test:
            editUser:
              headers: |
                "Authorization": "Bearer " + token
              before: |
                console.log('just a test!')
              uri-parameters: 
                id: 1
              body: 
                name: mark
                email: mark@gmail.com
              after: |
                assert.equal(res.body.name, "Andy")
```