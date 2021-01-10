import CardContents from '@ifca-root/react-component/src/components/CardList/CardContents'
import MainHeader from '@ifca-root/react-component/src/components/Header/MainHeader'
import SubHeaderAction from '@ifca-root/react-component/src/components/Header/SubHeaderAction'
import { List, MenuItem, Switch, TextField } from '@material-ui/core'
import {
  KeyboardDatePicker,
  MuiPickersUtilsProvider,
} from '@material-ui/pickers'
import { InputUpload } from '@ifca-root/react-component/src/components/Input/InputUpload'
import AppContext from 'containers/App/Store/AppContext'
import React, {
  ChangeEvent,
  Reducer,
  useContext,
  useReducer,
  useState,
} from 'react'
import { useHistory, useLocation, useParams } from 'react-router'
import { Controller, useForm } from 'react-hook-form'
import {
  useGetSoftware2Query,
  useCreateSubscriptionMutation,
  useGetAccountQuery,
  useUpdateSubscriptionMutation,
  GetAccountDocument,
} from 'generated/graphql'
import DateFnsUtils from '@date-io/date-fns'
import { IAction } from 'models'

interface ISubscriptionForm {
  softwareID: string
  salesOrderNo: string
  saleOrderDate: Date
  remark: string
  startDate: Date
  endDate: Date
  amount: string
  currencyID: string
  noOfUser: string
  terminationReason: string
  entityControl: {
    noOfEntity: Number
    noofExtraEntity: Number
  }
}

interface UserProps {
  isTerminated: boolean
}

export const SubscriptionForm = (props: any) => {
  let location = useLocation()
  const editData: any = location?.state
  console.log('data', editData)

  const handleTerminateForm = mode => {
    if (mode === 'edit') {
      return 'block'
    } else {
      return 'none'
    }
  }

  const handleTerminateForm2 = mode => {
    if (mode === 'edit') {
      return true
    } else {
      return false
    }
  }

  const { mode } = props

  const initialState: UserProps = {
    isTerminated: mode === 'add' ? false : editData?.data?.isTerminated,
  }

  const { globalState, dispatch } = useContext(AppContext as any)
  let history = useHistory()

 
  let { accountID, subscribedID } = useParams()
  // console.log('ID', ID)

  const { data: { getAccount } = { getAccount: [] } } = useGetAccountQuery({
    variables: { ID: accountID },
  })

  const reducer: Reducer<UserProps, IAction> = (state, action) => {
    switch (action.type) {
      case 'reset':
        return initialState
      default:
        return { ...state, [action.type]: action.payload }
    }
  }
  const [state, deliver] = useReducer(reducer, initialState)

 
  const [
    createSubscription,
    { loading: mutationLoading, error: mutationError },
  ] = useCreateSubscriptionMutation({
    onError: error => {
      console.log('ERROR', error)
    },
    onCompleted: data => {
      history.push({
        pathname: `/account/submenu/${editData?.data?.ID}/subscribed`,
        state: location.state,
      })
      console.log(history)
    },
  })

  const [
    updateSubscription,
    { loading: updateLoading, error: updateError },
  ] = useUpdateSubscriptionMutation({
    onError: error => {
      console.log('ERROR', error)
    },
    onCompleted: data => {
      history.push({
        pathname: `/account/submenu/${editData?.data?.ID}/subscribed`,
        state: location.state,
      })
      console.log(history)
    },
  })

  // const softwareName = editData?.data?.subscription[editData?.index].software
  // console.log('something', softwareName)

  const { handleSubmit, register, errors, control, reset } = useForm<
    ISubscriptionForm
  >({
    defaultValues: {
     
    },
  })

 

  const onSubmit = data => {
    console.log('data', data)
    console.log('state', location.state)
    mode === 'add'
      ? createSubscription({
          variables: {
            input: {
              accountID: accountID,
              softwareID: data.softwareID,
              subscriptionAmt: Number(data.amount),
              startDate: data.startDate,
              endDate: data.endDate,
              noOfUser: Number(data.noOfUser),
              saleOrderDate: data.saleOrderDate,
              salesOrderNo: data.salesOrderNo,
              remark: data.remark,
              entityControl: {
                noOfEntity: Number(data.noOfContract),
                noofExtraEntity: Number(data.noOfExtraEntity),
              },
            },
          },
          refetchQueries: [
            {
              query: GetAccountDocument,
              variables: {
                accountID: accountID,
              },
            },
          ],
        })
      : updateSubscription({
          variables: {
            input: {
              ID: subscribedID,
              accountID: accountID,
              softwareID: data.softwareID,

              subscriptionAmt: Number(data.amount),
              startDate: data.startDate,

              endDate: data.endDate,
              noOfUser: Number(data.noOfUser),
              saleOrderDate: data.saleOrderDate,

              salesOrderNo: data.salesOrderNo,

              remark: data.remark,
              isTerminated: state.isTerminated,
              terminationReason: data.accountTermination,
              entityControl: {
                noOfEntity: Number(data.noOfContract),
                noofExtraEntity: Number(data.noOfExtraEntity),
              },
            },
          },
          refetchQueries: [
            {
              query: GetAccountDocument,
              variables: {
                ID: accountID,
              },
            },
          ],
        })
  }

 

  const { data: { getSoftware } = { getSoftware: [] } } = useGetSoftware2Query()

  const [contracts, setContracts] = useState()

 

  const [upload, setUpload] = useState('')
  const handleUploadChange = (event: ChangeEvent<HTMLInputElement>) => {
    setUpload(event.target.value)
  }

  const handleUpload = e => {
    setUpload(e.target.files[0]?.name)

    console.log('file', e.target.files[0]?.name)
  }

  const [selectedDate, setSelectedDate] = useState(new Date())

  

  const handleDateChange = (date: Date | null) => {
    setSelectedDate(date)
  }

  console.log(selectedDate)

  const modeText = mode === 'edit' ? 'Edit' : 'New'

  return (
    <>
      <MainHeader
        mainBtn="back"
        onClick={() =>
          history.push({
            pathname: `/account/submenu/${editData?.data?.ID}/subscribed`,
            state: getAccount[0],
          })
        }
        smTitle="Accounts"
        title={getAccount[0]?.name}
        routeSegments={[
          { name: 'Main' },
          { name: 'Accounts' },
          { name: 'Submenu' },
          { name: 'Subscribed Product' },
          { name: `${modeText}`, current: true },
        ]}
      />
      <List className="core-list subheader product-form">
        <form onSubmit={handleSubmit(onSubmit)} id="submit-form">
          <SubHeaderAction
            title={`${modeText} Subscribed Product`}
            actionTitle="SAVE"
            // action={() =>
            //   history.push({
            //     pathname: `/account/submenu/${editData?.data?.ID}/subscribed`,
            //     state: location.state,
            //   })
            // }
          />

          <CardContents>
            <Controller
              as={
                <TextField onChange={handleChange}>
                  {getSoftware.map(product => (
                    <MenuItem
                      key={product.name}
                      // value passing the ID
                      value={product.ID}
                    >
                      {product.name}
                    </MenuItem>
                  ))}
                </TextField>
              }
              fullWidth
              id="standard-select-currency"
              select
              required
              name="softwareID"
              label="Product Name"
              // margin="normal"
              //defaultValue={''}
              defaultValue={
                mode === 'edit'
                  ? getAccount[0]?.subscription[editData?.index]?.software.ID
                  : ''
              }
              // defaultValue={mode === 'edit' ? getAccount[0]?.currencyID : ''}

              value={contracts}
              inputRef={register({})}
              control={control}
            />

            <Controller
              as={TextField}
              required
              fullWidth
              label="Sales Order No."
              name="salesOrderNo"
              defaultValue={
                mode === 'edit'
                  ? getAccount[0]?.subscription[editData.index]?.salesOrderNo
                  : ''
              }
              control={control}
              inputRef={register({})}
            />

            <MuiPickersUtilsProvider utils={DateFnsUtils}>
              <Controller
                as={KeyboardDatePicker}
                clearable
                // onChange={(date: Date | null) => {
                //   console.log(date)
                // }}
                format="dd MMM yyyy"
                margin="normal"
                id="date-picker-dialog"
                label="Sales order Date"
                //disableToolbar
                value={new Date()}
                KeyboardButtonProps={{
                  'aria-label': 'change date',
                }}
                defaultValue={
                  mode === 'edit'
                    ? getAccount[0]?.subscription[editData.index]?.saleOrderDate
                    : new Date()
                }
                fullWidth
                required
                allowKeyboardControl
                name="saleOrderDate"
                control={control}
                inputRef={register({})}
              />
            </MuiPickersUtilsProvider>

            <Controller
              as={TextField}
              required
              fullWidth
              label="Remark"
              name="remark"
              defaultValue={
                mode === 'edit'
                  ? getAccount[0]?.subscription[editData.index]?.remark
                  : ''
              }
              control={control}
              inputRef={register({})}
            />

            <MuiPickersUtilsProvider utils={DateFnsUtils}>
              <Controller
                as={KeyboardDatePicker}
                clearable
                //onChange={handleDateChange}
                //variant="inline"
                format="dd MMM yyyy"
                margin="normal"
                id="date-picker-dialog"
                label="Start Date"
                //disableToolbar
                value={new Date()}
                KeyboardButtonProps={{
                  'aria-label': 'change date',
                }}
                className="right"
                defaultValue={
                  mode === 'edit'
                    ? getAccount[0]?.subscription[editData.index]?.startDate
                    : new Date()
                }
                fullWidth
                required
                allowKeyboardControl
                name="startDate"
                control={control}
                inputRef={register({})}
              />
            </MuiPickersUtilsProvider>

            <MuiPickersUtilsProvider utils={DateFnsUtils}>
              <Controller
                as={KeyboardDatePicker}
                clearable
                //onChange={handleDateChange}
                // variant="inline"
                format="dd MMM yyyy"
                margin="normal"
                id="date-picker-dialog"
                label="End Date"
                //disableToolbar
                value={selectedDate}
                KeyboardButtonProps={{
                  'aria-label': 'change date',
                }}
                fullWidth
                className="left"
                defaultValue={
                  mode === 'edit'
                    ? getAccount[0]?.subscription[editData.index]?.endDate
                    : new Date()
                }
                required
                allowKeyboardControl
                name="endDate"
                control={control}
                inputRef={register({})}
              />
            </MuiPickersUtilsProvider>

            <Controller
              as={TextField}
              required
              fullWidth
              label="Amount"
              name="amount"
              defaultValue={
                mode === 'edit'
                  ? getAccount[0]?.subscription[editData?.index]
                      ?.subscriptionAmt
                  : ''
              }
              control={control}
              inputRef={register({})}
            />

            <Controller
              as={TextField}
              required
              fullWidth
              label="No. of Contract"
              name="noOfContract"
              control={control}
              defaultValue={
                mode === 'add'
                  ? ''
                  : editData?.data?.subscription[editData?.index]?.entityControl
                      ?.noOfEntity
              }
              inputRef={register({})}
            />

            <Controller
              as={TextField}
              required
              fullWidth
              label="No. of Entity Project"
              name="noOfExtraEntity"
              control={control}
              defaultValue={
                mode === 'add'
                  ? ''
                  : editData?.data?.subscription[editData?.index]?.entityControl
                      ?.noofExtraEntity
              }
              inputRef={register({})}
            />

            <Controller
              as={TextField}
              required
              fullWidth
              label="No. of User"
              name="noOfUser"
              defaultValue={
                mode === 'edit'
                  ? getAccount[0]?.subscription[editData?.index]?.noOfUser
                  : ''
              }
              control={control}
              inputRef={register({})}
            />

            <InputUpload
              value={upload}
              label="Upload Attachment(s)*"
              onChange={handleUploadChange}
              handleUpload={e => handleUpload(e)}
            />

            <div style={{ display: handleTerminateForm(mode) }}>
              <span className="desc flex-space color-red">
                Terminate Account
              </span>
              <span className="desc">
                <Switch
                  checked={state.isTerminated}
                  onChange={(event: React.ChangeEvent<HTMLInputElement>) => {
                    deliver({
                      type: 'isTerminated',
                      payload: event.target.checked,
                    })
                  }}
                  name="isTerminated"
                  color="primary"
                />
              </span>

              <Controller
                as={TextField}
                required={handleTerminateForm2(mode)}
                fullWidth
                label="Termination Reason"
                name="terminationReason"
                control={control}
                inputRef={register({})}
              />
            </div>
          </CardContents>
        </form>
      </List>
    </>
  )
}
