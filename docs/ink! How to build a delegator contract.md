The first thing is encoding the user-defined struct in off-chain node.js, it will be like this:
```node.js
...
/// encode user-defined data with ink api.
/// the `{"name": "Nika", "age": 18, "phones": ["123", "456"]}, "someone", 666` is user-defined parameters, there're three here
const calleeEncode = calleeABI.findMessage('encode_user_multi_params').toU8a([{"name": "Nika", "age": 18, "phones": ["123", "456"]}, "someone", 666]);

// it cannot be used until now, we should make more encoding manually
let ecdStr = '0x';
for (let i = 1; i < calleeEncode.length; ++i){
  let stemp = calleeEncode[i].toString(16);
  if (stemp.length < 2){
    stemp = '0' + stemp;
  }
  ecdStr += stemp;
}

...
/// Next, we can call *delegator contract* to make a delegate call to any callee contract.
/// the `contract` is the *delegator contract* instance
/// the `callToContracts` is the proxy call method
/// `5CgMjHnZm7VAi8x9HrB4b8FXYytnUj1pqNUH92yUmY9A7g8C` is the callee contract address
/// `ecdStr` is the encoded parameters and related method selector of the user-defined *callee contract*
const { gasConsumed, result, output } = await contract.query['callToContracts'](sender.address, {value, gasLimit }, 
                                       "5CgMjHnZm7VAi8x9HrB4b8FXYytnUj1pqNUH92yUmY9A7g8C", ecdStr);
```

The second thing is the related *delegator contract* method
```Rust
...
/// because in our case we have more than one parameters for callee, so we need a `Wrapper` manually:
 struct Wrapper {
        data: ink_prelude::vec::Vec::<u8>,
    }
    
    impl Wrapper {
        pub fn new(data: ink_prelude::vec::Vec::<u8>) -> Self {
            Self {
                data,
            }
        }
    }
    
    impl scale::Encode for Wrapper {
        #[inline]
        fn size_hint(&self) -> usize {
            scale::Encode::size_hint(&self.data)
        }
    
        #[inline]
        fn encode_to<O: scale::Output + ?Sized>(&self, output: &mut O) {
            output.write(&self.data);
        }
    }

...
/// this is the delegate call
fn call_to_contracts(& self, callee_account: AccountId, msg: ink_prelude::vec::Vec::<u8>) -> ink_prelude::string::String{
            let data: ink_prelude::vec::Vec::<u8> = msg.clone().drain(4..).collect();
            let wrapped_data = Wrapper::new(data);
            
            let my_return_value: ink_prelude::string::String =  ink_env::call::build_call::<ink_env::DefaultEnvironment>()
                .call_type(
                    ink_env::call::Call::new()
                        .callee(callee_account)
                        .gas_limit(0)
                        .transferred_value(0))
                .exec_input(
                    ink_env::call::ExecutionInput::new(ink_env::call::Selector::new([msg[0], msg[1], msg[2], msg[3]]))
                    .push_arg(wrapped_data)
                )
                .returns::<ink_prelude::string::String>()
                .fire()
                .unwrap();
            my_return_value
        }
...
```
