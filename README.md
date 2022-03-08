## 1. immer

* 불변성

    - 정의 : 어떤 값을 직접적으로 변경하지말고 새롭게 만들어 내는것. 주소 자체가 달라지게 생성함. call by referene
        
    - example : 아래 예제는 spread operator를 사용하여 불변성을 지킴.
        
              const user2021 = { name : 'KMH', age : 26 }; // {name: 'KMH', age: 26}
            
              const user2022 = { ...user2021, age : user2021.age + 1} // {name: 'KMH', age: 27}

    - 하지만 위와 다르게 객체의 레벨이 깊어지게 되면, spread operator는 얕은 복사이기 떄문에 불변성이 꺠진다. 그렇기에 immer를 사용한다.

     * 사용법 
     
          produce(수정할 객체, 수정 로직이 담긴 함수)
       
          example : https://codesandbox.io/s/pedantic-grass-ojocz?fontsize=14
          
              import produce from 'immer';
              const state = {
                number: 1,
                dontChangeMe: 2
              };

              const nextState = produce(state, draft => {
                draft.number += 1;
              });
    

## 2. recoil

   * Atoms
     
       - 정의 : 상태의 단위이며 업데이트와 구독이 가능.
         
       - 업데이트 : 각각의 구독된 컴포넌트는 새로운 값을 반영하여 다시 렌더링 되어짐.(동일한 Atom이 여러 컴포넌트에서 사용되는 경우 모든 컴포넌트는 상태를 공유)
             
       -  useRecoilState 라는 훅을 이용하여 컴포넌트에서 Atom을 읽고 사용하며, 컴포넌트간에 상태가 공유 될 수 있다.
             
       -  한 컴포넌트에서 A라는 Atom을 수정하면 그 Atom을 사용하는 모든 컴포넌트의 Atom값이 모두 수정한 값으로 변한다.
             
       - hook : const [fontSize, setFontSize] = useRecoilState(fontSizeState);
           
       example :    
       
            const fontSizeState = atom({
              key: 'fontSizeState',
              default: 14,
            });

   * Selector
     
       - 정의 : Atom 또는 다른 Selector를 입력으로 받아들이는 순수 함수(동일한 파라미터가 들어오면 언제든지 항상 동일한 결과를 리턴하며 외부의 값 변경 X, 참조는 가능).
       
       - 상위의 Atom 또는 Selector가 업데이트 되면 하위의 Selector 함수도 재 실행.
             
       - 컴포넌트들은 Selector를 atom처럼 구독할 수 있으며 Selector가 변경되면 컴포넌트들도 다시 렌더링.
             
       - useRecoilValue 라는 훅을 사용함.
             
       - hook : const fontSizeLabel = useRecoilValue(fontSizeLabelState);
            
         example : 
         
             const fontSizeLabelState = selector({
               key: 'fontSizeLabelState',
               get: ({get}) => {
                 const fontSize = get(fontSizeState);
                 const unit = 'px';
             
                 return `${fontSize}${unit}`;
               },
             });;

   * redux, recoil 코드를 본 느낌
      - redux에서는 간단한 기능이여도 의무적으로 작성해야하는 코드들(보일러 플레이트)로인해 코드가 길었다. 그에 비해 recoil은 상당히 심플한편.

## 3. react-hook-form

  - 용도 : form을 더 편하게 사용하기 위한 라이브러리.
       - 타입스크립트로 작성된 라이브러리.
       - 불필요한 렌더링을 피하기위해 uncontrolled components를 사용하도록 설계
       
    example : 
    
        const {
           register,
           handleSubmit,
           formState: { errors }
         } = useForm();         
  - register : input 도는  select, combo box등을 등록하고 유효성 규칙을 적용. register("exampleRequired", { required: true })
  - handleSubmit : onSubmit
    
  - 참조 : https://react-hook-form.com/get-started, https://codesandbox.io/s/react-hook-form-v7-original-example-i4dxw?file=/src/LoginForm.jsx


## 4. react-query

  - 용도 : 비동기 로직을 처리하는데 사용
  - isLoading, isError, refetch, 데이터 캐싱등을 지원
  - react-query를 사용하려면 최상단에서 <QueryClientProvider client={queryClient}>로 감싸야함.(보통 App.js)
 
  ### * useQuery
    
    const { data, isLoading, error } = useQuery(queryKey(String || Array), queryFn(Promise), options(Object))

    queryKey : 해당 키값을 기준으로 데이터 캐싱을 관리하며, String || Array 
          - 쿼리에 변수를 사용하면 queryKey에도 변수가 있어야함
          - const { data, isLoading, error } = useQuery(['todos', id], () => axios.get(`http://.../${id}`));

    queryFn : promise를 반환하는 함수를 넣어야함
          - () => fetchTodo()
          - async () => {const data = await fetchTodoByName('kmh'); return data;}
  
    options (자주 사용하는것 위주)
          * stale이란 최신화가 필요한 데이터.  staleTime으로 설정한 시간만큼 fresh 상태가 유지됨. staleTime이 길어도, cacheTime이 짧으면 데이터는 금방 삭제됨.
          - enabled(boolean) : 쿼리가 자동으로 실행되지않게 설정.  {enabled : !!id} 해당 옵션은 id가 있을떄만 쿼리실행
          - retry (boolean | number | (failureCount: number, error: TError) => boolean) 실패한 쿼리를 재시도 하는 옵션
          - staleTime (number | Infinity) 데이터가 fresh 상태로 유지되는 시간.  해당 시간이 지나면 stale 상태가 됨. default 0
          - cacheTime (number | Infinity) inactive 상태인 캐시 데이터가 메모리에 남아있는 시간. 이 시간이 지나면 가비지 컬렉터에 의해 제거됨.
          - refetchOnMount (boolean | "always") 데이터가 stale 상태일 경우 마운트 시 마다 refetch를 실행
          - refetchOnWindowFocus (boolean | "always")  데이터가 stale 상태일 경우 윈도우 포커싱 될 때 마다 refetch를 실행하는 옵션
          - refetchOnReconnect (boolean | "always") 데이터가 stale 상태일 경우  재연결 될때 refetch 함

          - onSuccess ((data: TDdata) => void) 쿼리 성공
          - onError ((error: TError) => void) 쿼리 실패
          - onSettled ((data?: TData, error?: TError) => void) 성공 || 실패 모두 실행
          - initialData (TData | () => TData) 쿼리 캐시의 초기 데이터로 설정
         
        * 참조 : https://react-query.tanstack.com/guides/queries

## 5. yup
    
  - 용도 : 자바스크립트 객체 스키마 유효성 검사. 쉽게 말하면 사용자 지정 유효성 검사
    
    example : 
    
        const SignUpSchema = Yup.object().shape({
          firstName: Yup.string()
          .min(2, "Too Short!")
          .max(50, "Too Long!")
          .required("Firstname is required"),
        });
    - 참조 : https://github.com/jquense/yup

## 6. Hook
    
  - useState :  1번쨰는 현재 상태값을 가진 변수, 2번쨰는 상태값을 변경하는데 사용되는 함수
      - const [currentInterval, setCurrentInterval] =  useState(maxInterval);
  - useEffect : 컴포넌트가 렌더링 이후의 할일(function)
      - useEffect를 컴포넌트 안에 선언하여, 컴포넌트 안에 어떤 props 또는 state에 접근 가능
      - 렌더링 이후에 매번 수행됨
    
    <br/>
    
      - cleanup effect : 리턴값으로 함수를 사용하면 해당 함수는 컴포넌트가 마운트 해제 될떄 실행됨.
           - useEffect는 매번 실행되는것이고 scope를 보면 매번 다른 effect일 것이고 매번 실행될떄 그 잔류물들을 정리하기위해 cleanup effet를 이용하는것으로 이해함.
    
    <br/>
      - useEffect가 모든 렌더링마다 실행되면 성능 저하가 발생할 수 있으므로, 2번쨰 인자에 해당 값이변할떄만 작동하도록 조건을 적용 할 수있음. 변수 또는 배열
    
  - 참조 : https://ko.reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update
